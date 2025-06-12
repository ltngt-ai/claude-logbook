# CLI Message Resolution Implementation

## Date: 2025-06-11

## Summary
Implemented automatic name resolution in the MindSwarm CLI message commands to convert user-friendly agent names/aliases to FQNs (Fully Qualified Names) before sending to the backend API.

## Problem
The user discovered that the CLI message send command required FQNs for the 'from' parameter, but users don't know their FQN. The backend API by design requires FQNs, so the solution was to make the frontend CLI resolve user-friendly names to FQNs.

## Solution Implemented

### 1. **Message Send Command** (`mindswarm message send`)
- Modified to resolve both `--from` and `--to` parameters
- Uses `client.resolve_recipient()` to convert aliases/names to FQNs
- Special handling for 'user' (and variants like 'human', 'me', 'cli-user')
- Shows resolution in dim text: `Resolved recipient 'project-manager' to 'db08f59c-111e-476e-abfb-50d933e2f817.pm_db08f59c'`
- Warnings shown if resolution fails but continues with original value

### 2. **Message Inbox Command** (`mindswarm message inbox`)
- Modified to resolve agent_id parameter to FQN
- Same resolution logic as send command
- Updates table title to show resolved FQN

### 3. **Message Clear Command** (`mindswarm message clear`)
- Modified to resolve agent_id parameter to FQN
- Same resolution logic as other commands

### 4. **Server-side Fix**
- Fixed `_handle_clear_messages` in main.py:
  - Changed from async `await mailbox.get_messages()` to sync `mailbox.get_all_mail()`
  - Fixed `archive_mail()` call to only pass message_id (not agent_fqn)

## Files Modified

1. `/home/deano/projects/mindswarm-cli-private/src/mindswarm_cli/commands/message_commands.py`:
   - Updated `send` command to resolve sender and recipient
   - Updated `inbox` command to resolve agent_id
   - Updated `clear` command to resolve agent_id

2. `/home/deano/projects/mindswarm-core-private/src/mindswarm/server/main.py`:
   - Fixed `_handle_clear_messages` to use correct mailbox API

## Testing Results

### Message Send with Resolution:
```bash
./mindswarm message send --from user --to project-manager --subject "Test Resolution" --body "Testing name resolution in CLI"

# Output:
Resolved recipient 'project-manager' to 'db08f59c-111e-476e-abfb-50d933e2f817.pm_db08f59c'
✅ Message Sent
From: user
To: db08f59c-111e-476e-abfb-50d933e2f817.pm_db08f59c
```

### Inbox with Resolution:
```bash
./mindswarm message inbox project-manager

# Output:
Resolved agent 'project-manager' to 'db08f59c-111e-476e-abfb-50d933e2f817.pm_db08f59c'
Inbox - db08f59c-111e-476e-abfb-50d933e2f817.pm_db08f59c
```

### Clear with Resolution:
```bash
./mindswarm message clear user --force

# Output:
✅ Cleared 1 messages
```

## Key Design Decisions

1. **Frontend Resolution**: The CLI resolves names to FQNs before calling the backend, maintaining the backend's requirement for FQNs
2. **Special Cases**: 'user' and its variants are treated specially and not resolved
3. **FQN Detection**: If a name already contains '.', it's assumed to be an FQN and not resolved
4. **Graceful Fallback**: If resolution fails, the original value is used with a warning
5. **User Feedback**: Resolution is shown to users so they understand what's happening

## Benefits

1. **User-Friendly**: Users can now use simple names like "project-manager" instead of complex FQNs
2. **Backward Compatible**: FQNs still work directly if provided
3. **Transparent**: Users see what names are resolved to
4. **Consistent**: All message commands now support name resolution

## Next Steps

- Consider adding similar resolution to other agent-related commands if needed
- Could add a `--no-resolve` flag for users who want to bypass resolution
- Could cache resolutions for performance in interactive sessions