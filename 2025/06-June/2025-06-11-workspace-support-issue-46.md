# 2025-06-11: Workspace Support in CLI (Issue #46)

## Summary
Added workspace support to the Mind-Swarm CLI to fix issue #46. Previously, test harness had to use direct API calls because the CLI didn't support workspaces.

## Changes Made

### CLI Updates
1. **New workspace command** with subcommands:
   - `mindswarm workspace list` - List all workspaces
   - `mindswarm workspace create` - Create new workspace
   - `mindswarm workspace activate` - Set workspace as active

2. **Updated project create command**:
   - Added `--workspace` / `-w` option
   - If not specified, uses the active workspace
   - Shows error if no active workspace found

3. **Client updates**:
   - Added `workspace_id` parameter to `create_project` method
   - Method now passes workspace_id to API

### Repository Management
- Merged workspace support directly to main (no PR needed in private repos)
- Also merged assistant-chat branch improvements
- Renamed mindswarm-cli to mindswarm-cli-private for future public/private separation
- Updated local git remote URLs

## Test Harness Can Now Use CLI
With these changes, Mind-SwarmSimpleTasks can now use the CLI instead of direct API calls:

```bash
# Create project with CLI (uses active workspace)
mindswarm project create --name "Test Project" --description "Test" --path "/path/to/project"

# Or specify workspace explicitly
mindswarm project create --name "Test Project" --description "Test" --workspace "workspace-id"
```

## Next Steps
- Update Mind-SwarmSimpleTasks test harness to use CLI commands instead of API calls
- Consider creating mindswarm_simple_tasks repo with snake_case naming