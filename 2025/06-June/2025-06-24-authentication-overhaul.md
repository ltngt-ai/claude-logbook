# 2025-06-24 - Authentication System Overhaul

## Summary
Major refactoring of the authentication system to fix multiple issues with identity confusion, session persistence, and WebSocket connection management.

## Key Accomplishments

### 1. Fixed Username/Email Identity Confusion (PR #251 - MERGED)
- **Problem**: Backend was using `session.username` (login identifier) as email address
- **Impact**: Created bogus mailbox registrations like "test@example.com" 
- **Solution**: Fixed to use `session.system_email` for proper MindSwarm addresses
- **Result**: Prevents "No identity set" errors after authentication

### 2. Implemented Ephemeral Authentication (PR #52 - MERGED)  
- **Changes**:
  - Removed all localStorage persistence from auth store
  - Sessions now match WebSocket lifecycle exactly
  - Improved security - no tokens stored in browser storage
  - Simplified MainApp initialization flow
- **Benefits**: Fixed duplicate auth requests and connection issues

### 3. WebSocket Cleanup for Rapid Reconnects (PR #252 - OPEN)
- **Issue**: User refreshing page quickly causes stale WebSocket mappings
- **Solution**: Clean up old WebSocket mappings before registering new ones
- **Impact**: Prevents AUTH responses going to dead connections

## Architecture Insights

The authentication flow now works as:
1. Frontend connects anonymously for login
2. Sends AUTH_REQUEST with username
3. Backend AUTH agent validates and responds to user's system_email
4. Auth notification service forwards response to WebSocket
5. Frontend disconnects anonymous connection
6. MainApp establishes new authenticated connection with token in URL

## Known Issues

Created issue #253 for AUTH duplicate login prevention:
- AUTH agent tracks sessions globally and prevents re-login
- Page refresh fails because AUTH thinks user is still logged in
- Workaround: Restart server to clear AUTH's memory
- Need to make AUTH smarter about session lifecycle

## Lessons Learned

1. **Ephemeral vs Persistent State**: Making auth fully ephemeral simplified many edge cases but exposed the AUTH agent's session tracking limitations

2. **WebSocket URL Authentication**: The backend requires auth token in WebSocket URL query params - can't just send it later

3. **Race Conditions**: Multiple race conditions exist between WebSocket lifecycle and auth flow - careful sequencing required

4. **Agent-First Complications**: The AUTH agent's AI-driven logic for preventing duplicate logins doesn't account for browser refresh patterns

## Next Steps

- Get PR #252 reviewed and merged for WebSocket cleanup
- Implement proper session management in AUTH agent (issue #253)
- Consider session restoration flow if users find re-login annoying

## Code References

### Key Files Modified
- `mindswarm-ui-private/src/store/auth.ts` - Removed localStorage, made ephemeral
- `mindswarm-ui-private/src/App.tsx` - Simplified auth flow, fixed race conditions
- `mindswarm-core-private/src/main.py` - WebSocket cleanup on rapid reconnects
- `mindswarm-core-private/src/services/auth_notification_service.py` - Fixed identity handling

### Test Coverage
- Added comprehensive auth store tests
- Verified WebSocket connection lifecycle
- Manual testing of rapid refresh scenarios

## Related Issues & PRs
- PR #251 (mindswarm-core): Fix username/email identity confusion - MERGED
- PR #52 (mindswarm-ui): Implement ephemeral authentication - MERGED  
- PR #252 (mindswarm-core): WebSocket cleanup for rapid reconnects - OPEN
- Issue #253 (mindswarm-core): AUTH duplicate login prevention

---

*Entry by Claude (AI Agent) <claude@ltngt.ai>*