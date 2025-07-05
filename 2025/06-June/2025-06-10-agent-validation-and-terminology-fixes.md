# 2025-01-10 Agent Validation and Terminology Fixes

## Summary
Fixed agent type validation errors and standardized terminology across Mind-Swarm. The backend now provides helpful error messages when invalid agent types are used, showing available types.

## Key Accomplishments

### 1. Fixed Agent Type Validation
- Modified server error handling to preserve detailed validation messages
- Added `mindswarm.agent.types` API endpoint to get available agent types
- Server now shows: "No agent template found for type 'a'. Available types: assistant, project_manager, research, software_engineer, task_manager, watcher"

### 2. Standardized Agent Type Names
- Updated all Mind-SwarmSimpleTasks test configurations from single-letter types to full names:
  - 'a' → 'assistant'
  - 'p' → 'project_manager'
  - 't' → 'task_manager'
  - 's' → 'software_engineer'
  - 'r' → 'research'
  - 'w' → 'watcher'

### 3. Terminology Consistency
- Replaced all "spawn" terminology with "create" across repositories
- Identified inconsistent use of "agent type" vs "agent template" (needs future cleanup)

## Current Issues

### 1. Project PathManager Issue
- Error: "Project not found or has no PathManager"
- Projects created via API don't have PathManagers properly initialized
- This prevents agents from being created, which prevents them from being registered

### 2. Agent Registration Issue
- Error: "Unknown recipient: 'test-assistant'"
- Agents aren't being registered because creation fails due to PathManager issue
- Need to fix the root cause (PathManager) first

## Technical Details

### Changes Made
1. **Server Handler Fix** (main.py:720-725):
   ```python
   except ValueError:
       # Re-raise ValueError directly to preserve the detailed error message
       raise
   except Exception as e:
       # For other exceptions, wrap with additional context
       raise ValueError(f"Failed to create agent: {str(e)}")
   ```

2. **New API Endpoint** (main.py:771-777):
   ```python
   async def _handle_get_agent_types(self, params: dict) -> dict:
       """Get available agent types from the agent manager."""
       if self.agent_manager and hasattr(self.agent_manager, 'template_registry'):
           types = self.agent_manager.template_registry.list_types()
           return {"types": types}
       else:
           return {"types": []}
   ```

## Todo List (Current State)

### High Priority - Completed ✅
- Fix path expansion in server to handle ~ properly
- Fix logging in run_server.sh
- Fix system project PathManager initialization
- Remove hardcoded aliases from mailbox system
- Implement agent registry with ID/name/alias mapping
- Add mail recipient validation with proper errors
- Create backend API for recipient lookup
- Write comprehensive unit tests for mail system
- Update system assistant to use proper name/ID
- Test the new mail system with assistant
- Update CLI to use new recipient resolution
- Debug why agent processor is being cancelled
- Update agent prompts to use explicit mail for user communication
- Create API endpoint for user to check mailbox and retrieve mail
- Remove channel system completely
- Ensure both structured and unstructured API responses work
- Update Mind-SwarmSimpleTasks tests to validate new agent-first model
- Replace all 'spawn' terminology with 'create' across ALL repositories
- Debug why agents are not being created in tests
- Add proper validation error messages for invalid agent types
- Complete refactoring to remove 'name' concept from agent registry
- Remove legacy hierarchical mailbox code

### High Priority - In Progress 🔄
- Fix Project PathManager initialization for API-created projects

### High Priority - Pending 📋
- Add watchdog iteration count to ai_loop to prevent infinite loops
- Update CLI to check user mailbox and display agent responses
- Write tests to verify explicit communication system works correctly
- Implement mail reply functionality in agents

### Medium Priority - Pending 📋
- Add debug mail viewer for all system mail
- Update frontend to handle mail-based communication
- Remove WorkspaceProjectService and use only ProjectManager
- Standardize terminology: use 'agent_type' instead of 'agent template'

## Next Steps
1. Fix the Project PathManager initialization issue
2. Verify agents can be created and registered properly
3. Run full test suite to ensure everything works
4. Create GitHub issues for remaining medium priority items

## Notes
- The validation errors are now much more helpful for users
- Tests are using correct agent type names
- Need to investigate why ProjectManager's get_path_manager isn't working for API-created projects