# GitHub Integration Implementation Guide - 2025-06-08

## Quick Start: Core Git Integration

### Step 1: Create Git Integration Module
```python
# src/mindswarm/integrations/git/__init__.py
from .repository import GitRepository
from .tools import GitTools
from .exceptions import GitError, MergeConflictError

__all__ = ['GitRepository', 'GitTools', 'GitError', 'MergeConflictError']
```

### Step 2: Implement GitRepository Class
```python
# src/mindswarm/integrations/git/repository.py
import asyncio
from pathlib import Path
from typing import List, Optional, Dict, Any
import git  # GitPython library
from mindswarm.core.exceptions import MindSwarmError

class GitRepository:
    """Manages Git operations for a MindSwarm project"""
    
    def __init__(self, project_path: Path):
        self.project_path = project_path
        self._repo = None
        
    @property
    def repo(self):
        """Lazy load Git repo"""
        if self._repo is None:
            if (self.project_path / '.git').exists():
                self._repo = git.Repo(self.project_path)
            else:
                raise GitError("No Git repository found")
        return self._repo
        
    async def init(self, initial_branch: str = "main") -> None:
        """Initialize Git repository"""
        self._repo = git.Repo.init(self.project_path, initial_branch=initial_branch)
        
    async def create_branch(self, branch_name: str, 
                           checkout: bool = True) -> str:
        """Create and optionally checkout a new branch"""
        new_branch = self.repo.create_head(branch_name)
        if checkout:
            new_branch.checkout()
        return branch_name
        
    async def safe_commit(self, message: str, 
                         files: Optional[List[str]] = None) -> str:
        """Commit with safety checks"""
        if files:
            self.repo.index.add(files)
        else:
            self.repo.index.add('.')  # Stage all changes
            
        # Check for staged changes
        if not self.repo.index.diff("HEAD"):
            return "No changes to commit"
            
        commit = self.repo.index.commit(message)
        return commit.hexsha
```

### Step 3: Create Git Tools for Agents
```python
# src/mindswarm/tools/git/git_status.py
from mindswarm.tools.base import Tool, ToolResult
from mindswarm.integrations.git import GitRepository

class GitStatusTool(Tool):
    """Check Git repository status"""
    
    name = "git_status"
    description = "Check the status of the Git repository"
    
    parameters_schema = {
        "type": "object",
        "properties": {
            "verbose": {
                "type": "boolean",
                "description": "Show detailed status information"
            }
        }
    }
    
    async def execute(self, parameters: dict, context) -> ToolResult:
        """Execute git status command"""
        try:
            project_path = context.path_manager.workspace_path
            git_repo = GitRepository(project_path)
            
            # Get status information
            status = {
                "branch": git_repo.repo.active_branch.name,
                "is_dirty": git_repo.repo.is_dirty(),
                "untracked_files": git_repo.repo.untracked_files,
                "modified_files": [item.a_path for item in git_repo.repo.index.diff(None)],
                "staged_files": [item.a_path for item in git_repo.repo.index.diff("HEAD")]
            }
            
            return ToolResult(
                success=True,
                data=status,
                message=self._format_status_message(status)
            )
            
        except Exception as e:
            return ToolResult(
                success=False,
                error=str(e),
                message=f"Failed to get Git status: {e}"
            )
```

### Step 4: Update Project Manager for Git Integration
```python
# src/mindswarm/services/workspace/project_manager.py (addition)

async def create_project_with_git(self, project_data: ProjectCreateNew) -> Project:
    """Create a new project with Git initialization"""
    # Create project as before
    project = await self.create_project(project_data)
    
    # Initialize Git if requested
    if project_data.git_enabled:
        git_repo = GitRepository(project.path)
        await git_repo.init()
        
        # Create initial .gitignore
        gitignore_content = """
# MindSwarm
.mindswarm/sessions/
.mindswarm/logs/
.mindswarm/tmp/
*.log

# Python
__pycache__/
*.py[cod]
*$py.class
.env
.venv/
venv/

# IDE
.vscode/
.idea/
*.swp
"""
        gitignore_path = project.path / '.gitignore'
        gitignore_path.write_text(gitignore_content)
        
        # Initial commit
        await git_repo.safe_commit("Initial commit - MindSwarm project setup")
        
    return project
```

## Plugin Architecture for GitHub

### Step 1: Create Plugin Base
```python
# src/mindswarm/plugins/base.py
from abc import ABC, abstractmethod
from typing import List, Dict, Any

class MindSwarmPlugin(ABC):
    """Base class for MindSwarm plugins"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """Plugin name"""
        pass
        
    @property
    @abstractmethod
    def version(self) -> str:
        """Plugin version"""
        pass
        
    @abstractmethod
    def get_tools(self) -> List[Tool]:
        """Return list of tools provided by plugin"""
        pass
        
    @abstractmethod
    async def initialize(self, config: Dict[str, Any]) -> None:
        """Initialize plugin with configuration"""
        pass
```

### Step 2: GitHub Plugin Structure
```
src/mindswarm/plugins/github/
├── __init__.py
├── plugin.py          # Main plugin class
├── auth.py           # Authentication handling
├── client.py         # GitHub API client
├── tools/
│   ├── __init__.py
│   ├── issues.py     # Issue management tools
│   ├── pulls.py      # Pull request tools
│   └── repos.py      # Repository tools
└── workflows/
    ├── __init__.py
    └── issue_driven.py
```

### Step 3: Implement GitHub Plugin
```python
# src/mindswarm/plugins/github/plugin.py
from mindswarm.plugins.base import MindSwarmPlugin
from .tools import get_github_tools
from .auth import GitHubAuthProvider

class GitHubPlugin(MindSwarmPlugin):
    """GitHub integration plugin"""
    
    name = "github"
    version = "1.0.0"
    
    def __init__(self):
        self.auth_provider = None
        self.tools = []
        
    async def initialize(self, config: Dict[str, Any]) -> None:
        """Initialize GitHub plugin"""
        # Set up authentication
        self.auth_provider = GitHubAuthProvider(
            method=config.get('auth_method', 'token'),
            credentials=config.get('credentials', {})
        )
        
        # Initialize tools with auth
        self.tools = get_github_tools(self.auth_provider)
        
    def get_tools(self) -> List[Tool]:
        """Return GitHub tools"""
        return self.tools
```

## Safe Workflow Implementation

### Branch Protection Strategy
```python
class AgentBranchManager:
    """Manages agent work branches"""
    
    BRANCH_PREFIX = "agent"
    
    @classmethod
    async def create_work_branch(cls, 
                                git_repo: GitRepository,
                                agent_id: str,
                                task_type: str) -> str:
        """Create isolated branch for agent work"""
        timestamp = datetime.now().strftime("%Y%m%d-%H%M%S")
        branch_name = f"{cls.BRANCH_PREFIX}/{task_type}-{agent_id}-{timestamp}"
        
        # Ensure we're on main/develop
        git_repo.repo.heads.main.checkout()
        
        # Create and checkout new branch
        await git_repo.create_branch(branch_name)
        
        return branch_name
        
    @classmethod
    async def create_pull_request(cls,
                                 git_repo: GitRepository,
                                 branch_name: str,
                                 title: str,
                                 description: str) -> Dict[str, Any]:
        """Create PR for agent work"""
        # This would use GitHub API
        # For now, return branch info
        return {
            "branch": branch_name,
            "title": title,
            "description": description,
            "status": "ready_for_review"
        }
```

### Automated Safety Checks
```python
class GitSafetyChecks:
    """Ensure safe Git operations"""
    
    @staticmethod
    async def pre_merge_checks(git_repo: GitRepository, 
                              source_branch: str,
                              target_branch: str) -> Dict[str, Any]:
        """Run safety checks before merge"""
        checks = {
            "conflicts": False,
            "tests_passing": True,
            "code_review": False,
            "security_scan": True
        }
        
        # Check for conflicts
        try:
            git_repo.repo.git.merge(source_branch, no_commit=True, no_ff=True)
            git_repo.repo.git.merge('--abort')
        except git.GitCommandError:
            checks["conflicts"] = True
            
        return checks
```

## Testing Strategy

### Unit Tests
```python
# tests/unit/integrations/git/test_repository.py
import pytest
from pathlib import Path
from mindswarm.integrations.git import GitRepository

@pytest.fixture
def temp_project(tmp_path):
    """Create temporary project directory"""
    project_dir = tmp_path / "test_project"
    project_dir.mkdir()
    return project_dir
    
async def test_git_init(temp_project):
    """Test Git repository initialization"""
    git_repo = GitRepository(temp_project)
    await git_repo.init()
    
    assert (temp_project / '.git').exists()
    assert git_repo.repo.active_branch.name == "main"
    
async def test_create_branch(temp_project):
    """Test branch creation"""
    git_repo = GitRepository(temp_project)
    await git_repo.init()
    
    branch_name = await git_repo.create_branch("feature/test")
    assert branch_name == "feature/test"
    assert git_repo.repo.active_branch.name == "feature/test"
```

### Integration Tests
```python
# tests/integration/test_github_workflow.py
async def test_issue_driven_development(project_with_git, github_plugin):
    """Test complete issue-driven development workflow"""
    # 1. Create issue
    issue = await github_plugin.create_issue(
        title="Add new feature",
        body="Implement user authentication"
    )
    
    # 2. Agent creates branch
    agent = Agent(name="developer")
    branch = await agent.execute_tool("git_create_branch", {
        "name": f"feature/issue-{issue.number}"
    })
    
    # 3. Agent makes changes
    await agent.execute_tool("create_file", {
        "path": "auth.py",
        "content": "# Authentication module"
    })
    
    # 4. Agent commits
    await agent.execute_tool("git_commit", {
        "message": f"Implement authentication (fixes #{issue.number})"
    })
    
    # 5. Agent creates PR
    pr = await agent.execute_tool("github_create_pr", {
        "title": f"Fix issue #{issue.number}",
        "head": branch,
        "base": "main"
    })
    
    assert pr.state == "open"
```

## Next Steps

1. **Implement Core Git Tools** (Week 1)
   - git_init, git_status, git_add, git_commit
   - git_branch, git_checkout, git_merge
   - git_diff, git_log

2. **Add Safety Features** (Week 2)
   - Branch protection rules
   - Automated conflict detection
   - Pre-commit hooks integration

3. **Build GitHub Plugin** (Week 3-4)
   - Authentication framework
   - Basic GitHub API tools
   - Rate limiting and caching

4. **Create Workflows** (Week 5-6)
   - Issue-driven development
   - Automated maintenance
   - Documentation sync

This implementation provides a solid foundation for Git/GitHub integration while maintaining MindSwarm's security and architectural principles.