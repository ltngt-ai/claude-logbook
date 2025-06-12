# 2025-06-11: Assistant Mail Processing Fix - Task Queue Removal

## Summary
Successfully fixed a critical tool calling protocol issue affecting both CLI and web frontend assistants. The root cause was unnecessary task queue abstraction that was violating OpenRouter's tool calling message sequence requirements.

## Problem Analysis
- **Issue**: Assistant worked once, then failed on subsequent messages with tool calling protocol error
- **Error**: `"Invalid parameter: messages with role 'tool' must be a response to a preceeding message with 'tool_calls'"`
- **Root Cause**: Mail processing was using a task queue that added multiple user messages to the same conversation context, leaving orphaned tool messages

## Architecture Issue
The agent manager was over-engineered with a task queue pattern:
```
Mail → Task Queue → Multiple user messages in same context → Tool protocol violation
```

This created conversation sequences like:
1. User message: "Process this mail..."
2. Assistant message: (with tool_calls to reply_mail)  
3. Tool message: (result of reply_mail)
4. User message: "Process this mail..." ← NEW MESSAGE - INVALID!

## Solution
Removed the task queue abstraction entirely for mail processing. Now:
```
Mail → Direct AI Loop → Complete conversation → Done
```

### Key Changes Made

#### 1. Agent Manager (`agent_manager.py`)
- **Before**: Mail queued as task via `session.task_queue.put()`
- **After**: Direct processing via `session.agent.process_message(prompt)`
- **Benefit**: Each mail is an isolated, complete conversation

#### 2. Reply Mail Tool (`reply_mail.py`)
- Added support for both calling conventions (arguments dict vs kwargs)
- Fixed parameter extraction to handle tool calling protocol properly

#### 3. Mailbox System (`mailbox.py`)
- Added `get_message(message_id)` method for reply_mail tool
- Enables proper message lookup for replies

#### 4. WebSocket User Registration (`main.py`)
- Added automatic user registration on WebSocket connection
- Proper cleanup on disconnect
- Generates UUIDs in format `user_{uuid}` for mailbox addressing

## Technical Details

### Conversation Flow (Fixed)
1. **Mail arrives** → Agent receives notification
2. **Direct processing** → `process_message()` called immediately  
3. **Complete AI loop** → User message → Assistant response → Tool calls → Tool results → Final response
4. **Context reset** → Next mail starts fresh conversation

### User Registration
- WebSocket users auto-registered as `user_{connection_id}`
- No more special 'user' aliases needed
- Clean mailbox addressing for frontend sessions

## Testing Results
- ✅ **CLI Assistant**: Multiple consecutive messages work perfectly
- ✅ **Web Frontend**: Multiple consecutive messages work perfectly  
- ✅ **Tool Protocol**: No more OpenRouter API errors
- ✅ **Message Flow**: Clean conversation sequences

## Architectural Improvement
This fix aligns with the "agent-first" principle mentioned in CLAUDE.md:
> "Everything is driven by intelligent AI agents with their own loops and context"

The task queue was artificial complexity. Agents should simply:
1. Receive mail
2. Process it with their AI loop  
3. Use tools as needed
4. Respond and sleep

No intermediate task abstraction needed.

## Files Modified
- `src/mindswarm/services/agents/agent_manager.py` - Removed task queue for mail
- `src/mindswarm/tools/communication/reply_mail.py` - Fixed calling conventions
- `src/mindswarm/core/communication/mailbox.py` - Added get_message method
- `src/mindswarm/server/main.py` - Added WebSocket user registration

## Next Steps
- Remove remaining 'user' special cases from backend
- Update CLI to handle new user registration system
- Both are lower priority cleanup tasks

## Commit
Branch: `fix-assistant-mail-processing`
Commit: `8a90fc9` - "fix: Remove task queue abstraction for assistant mail processing"

## Impact
This fix resolves a fundamental architectural issue and makes the assistant system robust for production use. Both CLI and web interfaces now handle multiple assistant interactions correctly without protocol violations.