# 2025-06-13: UI Agent CLI Conversation Implementation

## Summary

Implemented CLI conversation commands for natural language interaction with UI agents. Discovered and documented a fundamental architectural limitation: UI agents are session-based (WebSocket) while CLI uses one-shot requests.

## What Was Built

### 1. Conversation State Management Tools
- Created 5 individual tools in `/src/mindswarm/tools/conversation/`
- `start_conversation_tool.py`, `update_conversation_tool.py`, etc.
- Proper Mind-Swarm tool structure with state persistence

### 2. Dynamic UI Element Generation
- Comprehensive schemas in `/src/mindswarm/schemas/ui_elements.py`
- `generate_ui_elements_tool.py` with 15+ UI element types
- Predefined templates for common patterns

### 3. CLI Conversation Commands
```bash
mindswarm conversation start "message" [--conversation-id ID]
mindswarm conversation create-project "name" [-d description] [-i]
mindswarm conversation clone-github <url> [--name name]
```

### 4. Testing Infrastructure
- 3 simple tasks for UI agent project creation
- Test harness demonstrating the session limitation
- `test_ui_agent_persistence.py` proving the architectural mismatch

## Key Discovery: Session Architecture Mismatch

### The Problem
- **UI agents are spawned per WebSocket connection** (like web frontend)
- **CLI uses one-shot requests** - connect, request, disconnect
- **No persistence mechanism** for UI agents between CLI calls

### Evidence
```python
# This gets "agent_unavailable" error:
response = await client._send_request(
    'ui_request',
    'hello',
    {'message': 'Hello UI agent!'}
)
# Error: Agent ui_agents.ui_user_XXX is not available
```

### Root Cause
When WebSocket connects, server spawns UI agent:
```
INFO: Created new UI Agent session for user: user-6274751b
INFO: Spawned UI Agent for session user-6274751b: ui_user_627
```

But CLI's one-shot requests don't trigger this spawn.

## What Works vs What Doesn't

### ✅ Works
- Standard CLI commands via compatibility layer
- `mindswarm project create`, `workspace activate`, etc.
- Web interface conversations (maintains WebSocket)

### ❌ Doesn't Work  
- CLI conversation commands (no UI agent to talk to)
- Message response retrieval
- Conversation persistence/resumption

## Lessons Learned

1. **Agent-First Means Session-Based**: The UI agent architecture assumes persistent connections, not request-response patterns.

2. **User Registration Accumulation**: CLI registers users but never unregisters, causing orphaned mailboxes. Need cleanup mechanism.

3. **Compatibility Layer Still Valuable**: The old `project_request` style commands work because they bypass UI agents.

## Created GitHub Issue #124

Documented the architectural limitation with potential solutions:
1. WebSocket CLI mode (interactive session)
2. Server-side UI agent persistence
3. Lightweight CLI agents
4. Session token system

## Code Locations

- **Conversation Tools**: `/src/mindswarm/tools/conversation/`
- **UI Schemas**: `/src/mindswarm/schemas/ui_elements.py`  
- **CLI Commands**: `/src/mindswarm_cli/commands/conversation_commands.py`
- **Status Doc**: `/UI_AGENT_CLI_STATUS.md`
- **Test Script**: `/test_ui_agent_persistence.py`

## Next Steps

The conversation infrastructure is ready, waiting for one of:
- WebSocket support in CLI for persistent sessions
- Server-side UI agent hibernation/resumption
- Alternative architecture for CLI interactions

For now, standard CLI commands continue to work via the compatibility layer.