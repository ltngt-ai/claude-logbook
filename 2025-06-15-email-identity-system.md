# 2025-06-15: Email-Based Identity System & System Prompt Logging

## Overview

Implemented a major architectural improvement: email-based identity system for agents, replacing session-based identification. Also added system prompt logging to narrative logs for complete transparency.

## Key Accomplishments

### 1. Email-Based Identity System

**Problem Solved**: Frontend and backend had a bootstrap problem - frontend didn't have an identity until after registration, making it hard to send initial requests.

**Solution**: 
- Frontend generates/stores email address in localStorage on first visit
- Declares identity via `set_identity` message on WebSocket connection
- Backend maps email ↔ WebSocket connection
- All user state stored by email address (like cookies)

**Benefits**:
- No more anonymous users or registration dance
- User state persists across connections
- Clean, consistent identification throughout system

### 2. Generic Agent Template Variables

**Implementation**:
- Added `template_vars` parameter to agent creation
- Agents use `{{{variable}}}` placeholders in prompts
- Template variables resolved during prompt loading
- No special cases - works for any agent type

**Example**: UI agent prompt includes `{{{user_email}}}` which gets resolved to the actual user's email address.

### 3. Enhanced UI Agent Tools

**New Tools**:
- `UserContextTool`: Stores/retrieves user context by email address
- `set_current_workspace`: Sets workspace for a user
- `set_current_project`: Sets project for a user
- Updated `get_current_context` to use email-based storage

**Removed**:
- `register_user`: No longer needed with email-based identity
- `push_update`: Just send emails instead

### 4. System Prompt Logging

**Added**: `log_system_prompt()` to narrative logger
- Logs full system prompt during agent creation
- Shows resolved template variables
- Provides complete transparency into agent behavior

**Example Output**:
```
📋 SYSTEM PROMPT:
──────────────────────────────────────────────────
# UI Agent

I am the UI Agent, the intelligent interface between users and the MindSwarm agent ecosystem.

## My User Identity
I am specifically serving user: web-user-1749979052776-wfro3w@external.local.mindswarm.ltngt.ai
When executing tools that require user context, this email address identifies my user.
[...]
──────────────────────────────────────────────────
```

### 5. Bug Fixes

- **Agent Creation Collision**: Fixed by using MD5 hash for instance IDs instead of truncated user IDs
- **User ID Consistency**: Use full email address throughout, not just local part
- **Anti-Pattern Removal**: Deleted hardcoded `ui_agent_prompt.py` in favor of generic system

## Architecture Decisions

### Why Email-Based Identity?

1. **Simplicity**: Email addresses are unique, human-readable identifiers
2. **Stateless**: No server-side session management needed
3. **Familiar**: Like cookies but cleaner
4. **Agent-First**: Frontend is just another agent with an email address

### Why Template Variables?

1. **Generic**: Any agent can use dynamic context
2. **Clean**: No special cases or hardcoded prompts
3. **Extensible**: Easy to add new variables
4. **Transparent**: Variables visible in narrative logs

## Technical Details

### Frontend Changes
```typescript
// Generate/retrieve email on startup
const email = localStorage.getItem('mindswarm_email_address') || 
  `web-user-${Date.now()}-${randomId}@external.local.mindswarm.ltngt.ai`;

// Declare identity on connection
ws.send(JSON.stringify({
  type: 'set_identity',
  email_address: email
}));
```

### Backend Changes
```python
# Agent creation with template variables
await agent_manager.create_agent(
    project_id=project_id,
    agent_type="ui_agent",
    template_vars={'user_email': email_address}
)

# Prompt uses template
"I am specifically serving user: {{{user_email}}}"
```

## Lessons Learned

1. **Avoid Special Cases**: The original `ui_agent_prompt.py` was an anti-pattern. Generic systems are better.
2. **Identity First**: Establishing identity before operations simplifies everything
3. **Transparency Matters**: System prompt logging immediately helped debug issues
4. **Consistency Critical**: Using different user IDs in different parts caused collisions

## Next Steps

- Test the complete flow end-to-end
- Consider adding more template variables for other agent types
- Look into the reply_mail issue (agent generates response but doesn't send)

## PR Status

Updated PR #148 with all changes: https://github.com/ltngt-ai/mindswarm-core-private/pull/148