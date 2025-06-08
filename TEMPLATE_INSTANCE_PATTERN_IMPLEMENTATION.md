# Agent Template/Instance Pattern Implementation

Date: 2025-06-07

## Summary
Successfully implemented the MindSwarm agent template/instance pattern, allowing dynamic creation of agent instances from templates. This represents a major architectural shift from the static agent model to a flexible, scalable multi-agent system.

## Key Changes

### 1. Agent Template System
- Simplified agent configuration to 2 base templates: 'assistant' (a) and 'research' (r)
- Removed redundant `agent_templates.yaml` in favor of using `agents.yaml` directly
- Templates define agent capabilities, while instances have unique IDs

### 2. CLI Improvements
- Made parameters explicit: `--instance-id` and `--agent-type` for clarity
- Fixed agent listing to show all metadata (type, instance ID, state, etc.)
- Improved error messages to guide users

### 3. Server Updates
- Removed demo mode - server now uses real MindSwarm functionality
- Fixed API key loading from .env file with multiple path searches
- Added comprehensive logging for debugging

### 4. Mailbox System
- Updated to accept dynamic agent instance names (not just hardcoded aliases)
- Allows agents like 'research-agent', 'assistant-1', etc.

### 5. Interactive Mode Fixes
- Fixed project ID display in tables
- Added project selection by name or ID
- Improved error handling for invalid selections

## Technical Details

### Template Registry Pattern
```python
# Old: Direct agent ID reference
agent = agent_registry.get_agent('p')  # Patricia

# New: Template-based instance creation
session = await agent_manager.spawn_agent(
    agent_type='research',      # Template type
    instance_id='research-1'    # Unique instance
)
```

### Configuration Structure
```yaml
# agents.yaml - defines templates
agents:
  a:
    name: "Assistant"
    role: "assistant"
    description: "General-purpose AI assistant"
  r:
    name: "Research" 
    role: "research"
    description: "Research and analysis specialist"
```

### API Key Loading
- Added python-dotenv support
- Searches multiple paths for .env file
- Clear error messages when API key is missing

## Files Modified

### mindswarm-core-private
- `config/agents/agents.yaml` - Simplified to 2 agents
- `src/mindswarm/server/main.py` - Removed demo mode, fixed .env loading
- `src/mindswarm/services/agents/agent_manager.py` - Implemented template/instance pattern
- `src/mindswarm/services/agents/template_registry.py` - New file for template management
- `src/mindswarm/core/communication/mailbox.py` - Accept dynamic agent names
- `prompts/agents/a.prompt.md` - Simple assistant prompt
- `prompts/agents/r.prompt.md` - Simple research prompt

### mindswarm-cli
- `src/mindswarm_cli/commands/agent_commands.py` - Explicit parameters
- `src/mindswarm_cli/ui/interactive.py` - Fixed project display and selection
- `demo.sh` - Updated to use new parameter names

## Testing Results
- ✅ Unit tests: All 642 tests passing
- ✅ Server startup: Working with proper .env loading
- ✅ Project creation: Working via CLI
- ✅ Agent spawning: Successfully creates instances with unique IDs
- ✅ Agent listing: Shows all agent metadata correctly
- ✅ Interactive mode: Project selection working by ID or name

## Next Steps
1. Add more agent templates as needed
2. Implement agent communication between instances
3. Add resource tracking (CPU, memory placeholders currently)
4. Create integration tests for template/instance pattern
5. Update documentation with new patterns

## Lessons Learned
1. Explicit parameter names prevent confusion (instance-id vs type)
2. Supporting both ID and name selection improves UX
3. Dynamic systems need flexible validation (mailbox accepting any name)
4. Clear error messages are crucial for debugging

This implementation provides a solid foundation for the MindSwarm multi-agent system, allowing unlimited agent instances to be created dynamically from templates.