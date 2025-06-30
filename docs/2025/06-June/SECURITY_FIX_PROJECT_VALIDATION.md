# Security Fix: Project Validation for Agent and Task Operations

## Date: 2025-06-08

## Summary
Fixed a critical security vulnerability where agents and tasks could be created without valid projects, allowing unauthorized file system access and resource usage.

## The Security Issue

During testing of the continuation protocol (test 004), we discovered that agents were able to write files to the `mindswarm-core-private` directory instead of their designated project directory. Investigation revealed several critical security gaps:

1. **No Project Validation**: The server accepted any project_id without verifying it exists
2. **File System Access**: Agents without valid projects could still perform file operations
3. **Resource Exhaustion**: Unlimited agent spawning for non-existent projects
4. **Audit Trail Gap**: Operations could occur without proper project context

## Root Cause

The vulnerability existed in multiple layers:

### Server Layer (`src/mindswarm/server/main.py`)
- `_handle_spawn_agent()`: Accepted any project_id without validation
- `_handle_create_task()`: Created tasks for non-existent projects  
- `_handle_assign_task()`: Assigned tasks without project validation

### Agent Layer (`src/mindswarm/services/agents/agent_manager.py`)
- Only logged warnings when PathManager was missing
- Continued agent creation even without project validation

## The Fix

### 1. Server-Level Validation
Added security checks in all handlers:

```python
# In _handle_spawn_agent
if project_id != "default" and self.project_manager:
    project = self.project_manager.get_project(project_id)
    if not project:
        raise ValueError(f"Project not found: {project_id}")
```

### 2. Agent Manager Enforcement
Made PathManager requirement mandatory:

```python
# In AgentManager.create_agent_instance
if not project_path_manager and project_id != "default":
    raise ValueError(f"Project not found or has no PathManager: {project_id}")
```

### 3. File Tool Security
File tools already had proper checks:
- `ReadFileTool`: Raises `FileRestrictionError` without PathManager
- `WriteFileTool`: Raises `FileRestrictionError` without PathManager

## Security Impact

This fix prevents:
- Unauthorized agent spawning with fake project IDs
- File system access outside project boundaries
- Resource exhaustion attacks
- Bypass of project-based access controls

## Testing Insights

The issue was discovered when test 004 showed:
- Agent created files in `/home/deano/projects/mindswarm-core-private/`
- Warning: "No PathManager found for project test-project"
- Agent continued despite security violation

With the fix:
- Agents cannot be spawned for non-existent projects
- Tasks cannot be created/assigned without valid projects
- File operations fail safely with clear error messages

## Lessons Learned

1. **Security by Design**: Access control must be enforced at multiple layers
2. **Fail Secure**: Operations should fail when security context is unclear
3. **Test Isolation**: Tests revealed the vulnerability by attempting unauthorized operations
4. **Defense in Depth**: Multiple validation points prevent bypass attempts

## Future Considerations

1. Add rate limiting for project operations
2. Implement project access control lists (ACLs)
3. Add security audit logging for all project operations
4. Consider project capability restrictions (read-only, sandboxed, etc.)

## Code References

- Server validation: `src/mindswarm/server/main.py:321-327, 515-521, 577-584`
- Agent enforcement: `src/mindswarm/services/agents/agent_manager.py:333-335`
- File tool security: `src/mindswarm/tools/readonly_filesystem/read_file.py:95-98`