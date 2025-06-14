# 2025-06-14: UI Agent Mail Communication Fix

## Summary
Fixed critical issue where UI agents weren't replying to user messages despite successfully processing requests. Implemented model quirk handling for GPT-3.5's message ID truncation problem.

## The Problem
After implementing the ONE agent pattern with RFC 2822 email format:
- UI agents were waking on mail ✓
- UI agents were checking mail ✓  
- UI agents were processing requests (list_workspaces, etc.) ✓
- **UI agents were NOT replying to messages** ✗

## Root Cause Discovery
Through narrative logs, discovered that GPT-3.5-turbo was truncating RFC 2822 message IDs:
- Full ID: `<40e098cf-1e15-4809-9a24-e62fb6e8b75e@local.mindswarm.ltngt.ai>`
- Truncated by GPT-3.5: `<40e098cf-1e15-4809-9a24-e62fb6e8b75e@local.minds`
- Missing: `warm.ltngt.ai>`

This caused reply_mail to fail with "Original message not found".

## Solution: Model Quirks System

### 1. Added Quirk to model_capabilities.py
```python
"openai/gpt-3.5-turbo": {
    "quirks": {
        "truncates_long_tool_args": True  # GPT-3.5 truncates long strings in tool arguments
    }
}
```

### 2. Modified check_mail Tool
- Detects when agent uses problematic model via `_model_name`
- Provides shortened message IDs for those models: `<8charHash@ms>`
- Stores mapping between short and full IDs in message metadata

### 3. Updated reply_mail Tool  
- Handles both full RFC 2822 IDs and shortened IDs
- Searches by metadata for short ID mappings
- Maintains backward compatibility

### 4. Fixed AI Loop
- Added `_model_name` to tool execution context
- Fixed AIConfig field reference (model_id not model)

## Results
UI agents now successfully:
1. Receive mail with shortened IDs when using GPT-3.5
2. Use reply_mail without truncation issues
3. Deliver responses to users

## Discovered Issues
- **Config Bug**: Agents always use fallback model (GPT-3.5) instead of configured model
  - UI agent configured for Claude Haiku but uses GPT-3.5
  - Created GitHub issue #142
  - Medium priority - system functional with workaround

## Key Lessons
1. Model limitations require adaptive solutions
2. Narrative logs are invaluable for debugging agent behavior
3. The quirks system allows graceful degradation for less capable models
4. Sometimes the "bug" isn't in your code but in the model's behavior

## Code Changes
- `model_capabilities.py`: Added truncates_long_tool_args quirk
- `agent_narrative_logger.py`: Fixed to not truncate critical fields
- `mailbox.py`: Added generate_message_id function
- `check_mail.py`: Added short ID generation for problematic models
- `reply_mail.py`: Added short ID lookup capability
- `ai_loop.py`: Added model name to tool context

## Next Steps
- Fix config bug so agents use their configured models
- Continue with frontend/backend integration
- Consider moving UI agents to better models once config is fixed