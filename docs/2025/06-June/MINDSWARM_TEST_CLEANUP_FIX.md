# MindSwarm Test Framework Cleanup Fix

Date: 2025-06-08

## Issue

The MindSwarmSimpleTasks test framework had a critical issue where agents persisted between test runs, causing subsequent tests to fail with "agent already exists" errors. The root cause was that the `mindswarm agent terminate` command prompted for confirmation, breaking automated cleanup in test scripts.

## Investigation

1. **Initial Workaround**: The test scripts had implemented a server restart workaround to clean up agents, which was not ideal for a production system.

2. **Root Cause**: The `terminate` command in the CLI was using `click.confirm()` which prompted for user confirmation, making it impossible to use in automated scripts even with piped input.

3. **Server Analysis**: The server's terminate functionality was working correctly - it accepted both simple agent IDs and full FQNs, extracting the agent ID from the FQN when needed.

## Solution

### 1. CLI Enhancement
Added a `--yes/-y` flag to the `mindswarm agent terminate` command to skip confirmation:

```python
@agent.command()
@click.argument("agent_fqn")
@click.option("--force", "-f", is_flag=True, help="Force terminate without graceful shutdown")
@click.option("--yes", "-y", is_flag=True, help="Skip confirmation prompt")
@click.pass_context
def terminate(ctx: click.Context, agent_fqn: str, force: bool, yes: bool):
    """Terminate an agent."""
    client: MindSwarmClient = ctx.obj["client"]
    
    if not force and not yes:
        if not click.confirm(f"Are you sure you want to terminate agent '{agent_fqn}'?"):
            console.print("[yellow]Termination cancelled.[/yellow]")
            return
```

### 2. Test Framework Updates

#### test-harness.sh
- Removed workaround comments
- Updated `cleanup_existing_agents()` to use proper terminate with `-y` flag
- Updated `cleanup_test()` to terminate agents after each test

#### cleanup-agents.sh
- Replaced server restart workaround with proper agent termination
- Uses graceful termination first, then force if needed
- Properly handles JSON parsing when no agents exist
- Iterates through all agents and terminates them individually

#### run-all-tests.sh
- Added cleanup before starting tests
- Added cleanup after each test
- Added final cleanup at the end
- Includes proper PATH setup for mindswarm CLI

### 3. Server Fix
Removed the automatic `--demo` flag from `start_server.py` as the server no longer supports this flag.

## Testing

1. Spawned test agents and verified terminate works with `-y` flag
2. Ran full test cycle and confirmed proper cleanup between tests
3. Verified no agents persist after test completion
4. Tested both graceful and force termination modes

## Files Changed

### mindswarm-cli
- `src/mindswarm_cli/commands/agent_commands.py` - Added `-y` flag

### MindSwarmSimpleTasks
- `lib/test-harness.sh` - Updated cleanup functions
- `lib/cleanup-agents.sh` - Complete rewrite to use proper termination
- `run-all-tests.sh` - Added cleanup steps

### mindswarm-core-private
- `start_server.py` - Removed automatic --demo flag

## Key Learnings

1. **Automation First**: When building test frameworks, always consider automation requirements. Interactive prompts break automated workflows.

2. **Fix Root Causes**: While workarounds can be tempting, fixing the fundamental issue (adding the `-y` flag) is better than complex workarounds (server restarts).

3. **Test Isolation**: Proper cleanup between tests is critical for reliable test suites. Each test should start with a clean environment.

4. **FQN Flexibility**: The server's ability to accept both simple IDs and full FQNs made the solution more robust.

## Next Steps

With proper cleanup in place, the SimpleTasks framework can now be used reliably to test task execution implementation when that feature is added to MindSwarm.