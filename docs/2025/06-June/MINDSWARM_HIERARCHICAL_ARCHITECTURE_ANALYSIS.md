# Mind-Swarm Hierarchical Architecture Analysis

Date: 2025-01-07

## Current Problem
The original AIWhisperer design conflates global, project, task, and session concerns, causing architectural issues in the server-based model.

## Proposed Hierarchical Structure

### 1. Server/Global Level
**Purpose**: Server-wide state and configuration
**Location**: `~/.mindswarm/` (user's home directory)

**Contents**:
- `server_config.yaml` - Server configuration
- `providers/` - AI provider configurations and credentials
- `global_tools/` - Custom tools available to all projects
- `server.db` - Server state database (SQLite for simplicity)
- `logs/` - Server-level logs
- `metrics/` - Server performance metrics

**State Examples**:
- Registered AI providers and their configurations
- Global tool registry
- Server performance metrics
- User preferences
- License/authentication info

### 2. Project Level
**Purpose**: Workspace that agents collaborate on
**Location**: `<project_root>/.mindswarm/`

**Contents**:
- `project.yaml` - Project configuration
- `agents/` - Project-specific agent configurations
- `tools/` - Project-specific tools
- `context/` - Project context and memory
- `logs/` - Project activity logs
- `state.db` - Project state database

**State Examples**:
- Project metadata (name, description, created date)
- Active tasks and their status
- Project-level agent registry
- Project history and context
- Code analysis cache
- Project-specific tool configurations

### 3. Task/Team Level
**Purpose**: Specific work streams within a project
**Location**: `<project_root>/.mindswarm/tasks/<task_id>/`

**Contents**:
- `task.yaml` - Task configuration
- `team.yaml` - Team composition and agent assignments
- `context/` - Task-specific context
- `logs/` - Task execution logs
- `artifacts/` - Task outputs
- `state.json` - Task state

**State Examples**:
- Task definition and goals
- Team composition (which agents, their roles)
- Task progress and status
- Task-specific memory/context
- Inter-agent communications for this task

## Agent Identity Architecture

### Hierarchical Naming Scheme

**Full Qualified Name (FQN)**: `project.task.agent.instance`

Examples:
- `Frontend.UpdateDocs.patricia.0` - First Patricia in UpdateDocs task
- `Backend.TestCoverage.alice.0` - Alice working on test coverage
- `Frontend.UpdateDocs.patricia.1` - Second Patricia instance

### Contextual Resolution Rules

1. **Within Task Context**: Can use shortest unique identifier
   - If only one Patricia: `patricia` or `p`
   - If multiple Patricias: `p.0`, `p.1`

2. **Cross-Task Communication**: Requires task qualifier
   - `UpdateDocs.patricia` or `TestCoverage.alice`

3. **Cross-Project Communication**: Requires full qualification
   - `Frontend.UpdateDocs.patricia.0`

### Alias System
- User-friendly names map to FQNs
- `Patricia` → `*.*.patricia.0` (if unique)
- `Patricia0` → `*.*.patricia.0` (if multiple exist)
- `TestPatricia` → `*.TestCoverage.patricia.*`

## Implementation Recommendations

### 1. State Management Service
```python
class StateHierarchy:
    def __init__(self):
        self.server_state = ServerState()
        self.projects = {}  # project_id -> ProjectState
        self.tasks = {}     # task_id -> TaskState
    
    def get_agent_context(self, fqn: str) -> AgentContext:
        """Build context from all three levels"""
        project, task, agent, instance = self.parse_fqn(fqn)
        return AgentContext(
            server=self.server_state,
            project=self.projects[project],
            task=self.tasks[f"{project}.{task}"],
            agent_id=f"{agent}.{instance}"
        )
```

### 2. Agent Registry Enhancement
```python
class HierarchicalAgentRegistry:
    def resolve_agent(self, identifier: str, context: Context) -> str:
        """Resolve partial identifier to FQN based on context"""
        # If already FQN, return as-is
        if self.is_fqn(identifier):
            return identifier
        
        # Try contextual resolution
        candidates = self.find_matches(identifier, context)
        if len(candidates) == 1:
            return candidates[0]
        elif len(candidates) > 1:
            raise AmbiguousAgentError(f"Multiple matches for '{identifier}': {candidates}")
        else:
            raise AgentNotFoundError(f"No agent found matching '{identifier}'")
```

### 3. Project Manager Service
```python
class ProjectManager:
    def __init__(self, server_root: Path):
        self.server_root = server_root
        self.active_projects = {}
    
    def create_project(self, project_id: str, workspace: Path) -> Project:
        """Initialize project structure"""
        project_dir = workspace / ".mindswarm"
        project_dir.mkdir(exist_ok=True)
        
        # Create project structure
        (project_dir / "agents").mkdir(exist_ok=True)
        (project_dir / "tasks").mkdir(exist_ok=True)
        (project_dir / "context").mkdir(exist_ok=True)
        
        # Initialize project state
        project = Project(
            id=project_id,
            workspace=workspace,
            created_at=datetime.now()
        )
        self.active_projects[project_id] = project
        return project
```

### 4. Task Orchestrator
```python
class TaskOrchestrator:
    def create_task(self, project: Project, task_def: TaskDefinition) -> Task:
        """Create new task with team assignment"""
        task = Task(
            id=f"{project.id}.{task_def.name}",
            project=project,
            definition=task_def
        )
        
        # Spawn agent instances for team
        for agent_spec in task_def.team:
            agent_fqn = self.spawn_agent(
                project=project.id,
                task=task_def.name,
                agent_type=agent_spec.type,
                config=agent_spec.config
            )
            task.team.append(agent_fqn)
        
        return task
```

## Benefits of This Architecture

1. **Clear Separation of Concerns**
   - Server concerns (providers, global config)
   - Project concerns (workspace, project agents)
   - Task concerns (specific work, team coordination)

2. **Scalability**
   - Multiple projects simultaneously
   - Multiple tasks per project
   - Multiple agent instances per type

3. **Flexibility**
   - Scheduled tasks (nightly doc updates)
   - Parallel workstreams
   - Different agent configurations per task

4. **Maintainability**
   - Clear state boundaries
   - Predictable file locations
   - Hierarchical configuration inheritance

## Migration Path

1. **Phase 1**: Implement state hierarchy
   - Create StateHierarchy service
   - Update PathManager to understand levels
   - Implement hierarchical config loading

2. **Phase 2**: Update Agent System
   - Implement FQN naming
   - Update AgentRegistry for hierarchical lookup
   - Add agent spawning/cloning capability

3. **Phase 3**: Project & Task Management
   - Implement ProjectManager
   - Implement TaskOrchestrator
   - Update UI to show project/task structure

4. **Phase 4**: Update Tools & Context
   - Tools understand their execution context
   - Context isolation between tasks
   - Cross-task communication protocols

## Open Questions

1. **State Persistence**: SQLite per level or unified database?
2. **Configuration Inheritance**: How do task configs override project configs?
3. **Resource Limits**: How to limit agents per task/project?
4. **Cross-Project Communication**: Should agents communicate across projects?
5. **Task Lifecycle**: How are completed tasks archived?