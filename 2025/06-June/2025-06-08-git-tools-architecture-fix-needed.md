# Git Tools Architecture Fix Needed

## Date: 2025-06-08

### Summary
Discovered that Git tools need to be refactored to use the correct MindSwarm tool architecture pattern.

### Current Issues

1. **Incorrect Imports**
   - Git tools import: `from mindswarm.tools.base import Tool, ToolResult`
   - These classes don't exist
   - Should use: `from mindswarm.tools.base_tool import BaseTool`

2. **Wrong Base Class**
   - Git tools inherit from non-existent `Tool` class
   - Should inherit from `BaseTool`

3. **Different Return Pattern**
   - Git tools use `ToolResult` objects
   - Should return dictionaries with status/message

### Correct Pattern (from write_file tool)

```python
from mindswarm.tools.base_tool import BaseTool
from mindswarm.core.unified_logging import get_logger

class ToolName(BaseTool):
    @property
    def name(self) -> str:
        return "tool_name"
    
    @property
    def description(self) -> str:
        return "Tool description"
    
    @property
    def parameters_schema(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {...},
            "required": [...]
        }
    
    @property
    def category(self) -> Optional[str]:
        return "Category"
    
    @property
    def tags(self) -> List[str]:
        return ["tag1", "tag2"]
    
    def execute(self, **kwargs) -> Dict[str, Any]:
        # Get agent context from kwargs
        agent_context = kwargs.get('_agent_context')
        
        # Return dictionary with status/message
        return {
            "status": "success" | "error",
            "message": "Result message",
            # other data...
        }
    
    def get_ai_prompt_instructions(self) -> str:
        return "Instructions for AI"
```

### Tools That Need Fixing

1. **Original Git tools** (use Tool/ToolResult pattern):
   - git_status.py
   - git_init.py

2. **New Git tools** (use Tool/ToolExecutionContext pattern):
   - git_add.py
   - git_commit.py
   - git_branch.py
   - git_checkout.py
   - git_merge.py
   - git_diff.py
   - git_log.py
   - git_stash.py

### Verified Working

Created `git_add_fixed.py` following the correct pattern:
- Inherits from `BaseTool`
- Uses properties for metadata
- Returns dictionaries
- Gets agent context from kwargs
- Handles async Git operations with event loop

The fixed tool works correctly and follows MindSwarm conventions.

### Next Steps

1. Fix all Git tools to use correct architecture
2. Update tests to match new pattern
3. Ensure all tools have proper unit tests
4. Update __init__.py to import fixed tools
5. Remove old broken implementations

The Git integration with ProjectManager is working correctly, but individual Git tools need architectural alignment.