# 2025-06-23: Major Authentication Refactoring - Email to Username

## Overview
Completed a major refactoring of the Mind-Swarm authentication system to use "username" instead of "email" throughout. This change clarifies the distinction between login credentials (username) and internal Mind-Swarm mailbox addresses (system_email).

## Problem
The authentication system was confusing because:
- "email" was used for login credentials (often looks like an email)
- "system_email" was used for Mind-Swarm mailbox addresses
- This led to bugs and confusion about which email to use where
- Frontend was sending malformed RFC2822 addresses like "test@example.com@external.com"

## Solution
Systematically refactored the entire auth system:

### Backend Changes
1. **Models**: Updated `User` and `AuthenticatedSession` to use `username` field
2. **Storage**: Modified `UserStorage` to use `username` for all lookups
3. **Auth Tools**: Updated all tools to expect `username` parameter:
   - `validate_user_credentials`
   - `register_user`
   - `record_failed_attempt`
   - `create_user_session`
4. **AUTH Agent**: Updated prompt to expect `username` in requests
5. **Session Storage**: Changed schema from `email` to `username`
6. **JWT Tokens**: Now include `username` instead of `email`
7. **Auth Middleware**: Fixed to pass `username` to UserSession
8. **WebSocket Handler**: Fixed to register auth requests by username

### Frontend Changes
1. **LoginPage**: Send `username` instead of `email` in AUTH_REQUEST
2. **From Address**: Use `anonymous@client` instead of malformed email
3. **Auth Response Handling**: Added specific handler for `auth_response` message type
4. **Waiter Resolution**: Track what each waiter is waiting for to resolve correctly
5. **Identity Setting**: Set system_email after successful auth

## Key Insights
- AUTH agent was smart enough to reject malformed requests immediately
- The agent's context persistence led to overly cautious behavior after errors
- Better to have false negatives than false positives for auth
- Removing backward compatibility made the code much cleaner

## Challenges Overcome
1. **Auth Response Delivery**: Responses weren't reaching frontend because WebSocket wasn't registered with username
2. **Message Correlation**: Frontend expected replies but AUTH sends new messages
3. **Session Storage**: Still referenced old `email` field causing crashes
4. **Circular Dependencies**: Had to carefully update all interconnected components

## PRs Created
- Backend: https://github.com/ltngt-ai/mindswarm-core-private/pull/248
- Frontend: https://github.com/ltngt-ai/mindswarm-ui-private/pull/50

## Lessons Learned
1. When refactoring field names, search for ALL references including:
   - Direct field access (`.email`)
   - Dictionary keys (`["email"]`)
   - Schema definitions
   - Database columns
2. Test the full flow end-to-end, not just individual components
3. Agent context can lead to unexpected behavior - consider context trimming
4. Clean breaks are better than backward compatibility for pre-1.0 software

## Next Steps
- Implement context trimming for agents to prevent overly persistent memory
- Consider adding integration tests for full auth flow
- Update any documentation referring to email-based auth