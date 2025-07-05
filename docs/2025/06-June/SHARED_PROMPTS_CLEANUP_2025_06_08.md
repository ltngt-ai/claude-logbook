# Shared Prompts System Cleanup
**Date**: June 8, 2025

## Overview
Cleaned up and simplified the shared prompts system in Mind-Swarm to remove confusion, duplication, and model-specific references that agents wouldn't understand.

## Changes Made

### 1. Commented Out Debbie References
- Debbie agent doesn't exist yet in the system
- Commented out all Debbie-specific code in `websocket_session_manager.py`
- Added TODO comments for when Debbie is implemented

### 2. Removed Duplicate Content from core.md
- Response channel instructions were duplicated in both `core.md` and `channel_system.md`
- Removed the duplicate section from `core.md` since it's handled by the channel system feature
- Kept core.md focused on fundamental agent behavior

### 3. Made tool_guidelines.md Generic
- Previous version referenced specific tools that don't exist in Mind-Swarm
- Rewrote to provide general principles without naming specific tools
- Clarified the confusion about "one tool per step" vs "batch operations"
  - Now explains: Use one tool per step, but maximize each tool's capabilities

### 4. Simplified Continuation Protocol
**Before**: Had 3 files with confusing hierarchy
- `continuation_protocol.md` (didn't explain how to format)
- `continuation_protocol_gemini.md` (duplicate of non-structured)
- `structured_continuation.md` (for structured output models)

**After**: Simplified to 2 clear paths
- `continuation_protocol.md` - For models WITHOUT structured output (JSON in content)
- `continuation_protocol_structured.md` - For models WITH structured output

### 5. Updated Mailbox Protocol with Correct FQN
- Added proper FQN (Fully Qualified Name) format: `project.task.agent.instance`
- Documented addressing options:
  - Full FQN: `Frontend.UpdateDocs.patricia.0`
  - Short names within same task: `patricia` or `p`
  - Broadcast patterns: `Frontend.*`, `*`
- Updated examples to use real agent names from the system

### 6. Removed Model Capability References
- Agents don't know their own model capabilities
- Removed references to "structured output" and "model capabilities" from `debug_options.md`
- Now just tells agents to "follow the continuation protocol instructions provided"

## Key Insights

1. **Agents are model-agnostic**: They shouldn't know about structured output or model capabilities
2. **Two-path system**: The prompt system handles model differences by loading different files
3. **Keep it simple**: Having too many variations creates confusion

## File Structure After Cleanup

```
prompts/shared/
├── channel_system.md              # Non-structured response channels
├── channel_system_structured.md   # Structured response channels
├── continuation_protocol.md       # Non-structured continuation (JSON in content)
├── continuation_protocol_structured.md  # Structured continuation
├── core.md                       # Core behavior (no duplicates)
├── debug_options.md              # Debug behavior (no model references)
├── mailbox_protocol.md           # Updated with FQN format
├── tool_calling_format.md        # Tool usage format
├── tool_guidelines.md            # Generic tool principles
└── [other files...]
```

## How the System Works

When spawning an agent, the prompt system:
1. Checks if the model supports structured output
2. Loads appropriate versions of channel_system and continuation_protocol
3. The agent just follows whatever instructions it receives
4. No need for agents to understand their model's capabilities

This creates a clean separation between system knowledge (which files to load) and agent knowledge (following loaded instructions).