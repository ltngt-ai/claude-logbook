# 2025-06-10: Agent Naming System Refactor Complete

## Summary
Completed a major refactor of the agent naming system in MindSwarm to eliminate confusion between IDs, names, and aliases. The system now uses a cleaner, more consistent approach.

## Key Changes

### 1. Removed 'name' Concept
- Eliminated the redundant `name` field from `AgentRegistration`
- All human-friendly names are now just aliases (set of strings)
- Backend uses only `agent_id` and `aliases` for clarity

### 2. Fixed Variable Naming
- Renamed confusing `session_id` to `agent_instance_id` throughout agent_manager.py
- Much clearer what each variable represents
- User feedback: "Seems suspicious to me, why is the agent_id being set to the 'session_id'"

### 3. Agent Registry Updates
- Created new `agent_registry.py` with centralized agent ID/alias management
- Supports FQN (Fully Qualified Name) format: `project_id.agent_id`
- Removed `_name_index`, only using `_alias_index` now
- All lookups go through `resolve_identifier()` method

### 4. Mail System Integration
- Mailbox now validates recipients through agent registry
- Proper error messages for unknown recipients
- User feedback: "backend should error on trying sending a mail to an unknown recipient"
- Special handling for 'user' and CLI user IDs

### 5. System Assistant Fix
- System assistant now uses aliases like all other agents
- "SystemAssistant" is passed as an alias, not a special name
- Follows the same patterns as all other agents

## Technical Details

### Agent Registration Structure
```python
@dataclass
class AgentRegistration:
    agent_id: str  # Unique system ID
    aliases: Set[str] = field(default_factory=set)  # All aliases
    project_id: Optional[str] = None
    registered_at: datetime = field(default_factory=datetime.now)
    metadata: Dict[str, any] = field(default_factory=dict)
```

### ID Resolution Flow
1. User provides identifier (could be ID, alias, or FQN)
2. Registry checks if it's already an FQN
3. If project context provided, tries project-scoped lookup first
4. Falls back to global lookup
5. Returns FQN or raises error

### Mail Validation
- Mail system calls `_resolve_agent_name()` before sending
- Validates recipient exists in registry
- Returns proper error: "Unknown recipient: 'X'. Agent not found in registry."

## User Feedback Addressed
1. ✅ "We still have this general confusing about id and names"
2. ✅ "backend should error on trying sending a mail to an unknown recipient"
3. ✅ "You fell into the trap we are trying to solve! instance_id and name are different things"
4. ✅ "I think from a clarity pov except for the user, we don't use name. A name is an alias"
5. ✅ "Seems suspicious to me, why is the agent_id being set to the 'session_id'"

## Benefits
- Eliminates confusion between names, aliases, and IDs
- Consistent pattern: backend uses IDs, frontends convert aliases
- Better error messages for unknown recipients
- Clearer code with descriptive variable names
- Centralized agent registry for all lookups

## Next Steps
- Write comprehensive unit tests for mail system
- Test the new mail system with assistant
- Update CLI to use new recipient resolution
- Remove WorkspaceProjectService (duplicate project system)

## Files Modified
- `/src/mindswarm/core/agent_registry.py` (NEW)
- `/src/mindswarm/core/communication/mailbox.py`
- `/src/mindswarm/services/agents/agent_manager.py`
- `/src/mindswarm/server/system_init.py`
- `/src/mindswarm/server/main.py`
- `/src/mindswarm/services/workspace/project_manager.py`

## Commits
- "Refactor agent naming system: Remove 'name' concept and use only ID/aliases"
- "Add logs and data directories to .gitignore"
- "Improve mailbox system and add comprehensive tests"

## Mailbox System Improvements

### Cleanup
- Removed legacy hierarchical mailbox code from `src/mindswarm/communication/`
- This was unused code - all imports use `core.communication.mailbox`

### Comprehensive Test Suite
Created `tests/unit/core/test_mailbox_with_agent_registry.py` with 16 tests covering:
- Agent registration and alias resolution
- Mail validation with proper error messages
- FQN (Fully Qualified Name) support and project context
- Message priorities and statuses  
- Reply chains and conversation threads
- Archive functionality
- Notification handlers
- Case-insensitive alias matching
- User aliases (user, human, me, cli-user-*)
- Global agents (no project)

### Bug Fixes
- Fixed mailbox reset function to set `_mailbox_system = None` for proper testing
- Fixed conversation thread algorithm to search all inboxes for replies
- All tests now passing!

### Key Test Examples
```python
# Test unknown recipient error
with pytest.raises(ValueError) as exc_info:
    mailbox.send_mail(mail)
assert "Unknown recipient: 'unknown-agent'" in str(exc_info.value)

# Test alias resolution  
registry.register_agent(
    agent_id="patricia",
    aliases={"p", "planner", "patricia-planner"},
    project_id="project1"
)
# Can send to any alias
mailbox.send_mail(mail_to_planner, project_id="project1")

# Test FQN support
mail.to_agent = "frontend.alice"  # Specific project.agent
```

The mailbox system is now rock solid with proper recipient validation!