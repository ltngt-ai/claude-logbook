# GitHub Issue: GPT-4o Multi-Phase Task Completion Failure

## Title
GPT-4o fails to complete multi-phase tasks (stops after Phase 2 of 3)

## Description
GPT-4o consistently fails to complete all phases of multi-phase tasks when using the continuation protocol. The model completes Phases 1 and 2 successfully but terminates before executing Phase 3, even with explicit instructions to complete all phases.

## Steps to Reproduce
1. Run test 004-task-continuation-protocol with GPT-4o
2. Observe that the model creates `data.txt` (Phase 1) and `summary.txt` (Phase 2)
3. Note that `status.txt` (Phase 3) is never created
4. The model sets continuation status to TERMINATE after Phase 2

## Expected Behavior
The model should complete all three phases:
- Phase 1: Create data.txt
- Phase 2: Create summary.txt  
- Phase 3: Create status.txt with completion report

## Actual Behavior
- Phase 1: ✅ Creates data.txt
- Phase 2: ✅ Creates summary.txt
- Phase 3: ❌ Fails to create status.txt (terminates early)

## Investigation Summary
- Verified both structured and non-structured output paths work correctly
- Forced GPT-4o to use non-structured output (same as Claude-2) - still failed
- Created enhanced prompts with explicit phase completion requirements - still failed
- Claude-2.1 passes the same test successfully

## Root Cause
This appears to be an inherent characteristic of GPT-4o's task completion heuristics. The model considers the task "complete enough" after major work is done, even when additional phases are explicitly required.

## Workaround Options (Not Implemented Yet)
1. **System-level phase orchestration**: External management of phase transitions
2. **Explicit task chaining**: Break multi-phase tasks into separate task assignments
3. **Forced continuation**: Override TERMINATE status when phases remain incomplete
4. **Model selection**: Use Claude models for multi-phase tasks

## Impact
- Affects any multi-phase task execution with GPT-4o
- Limits the model's usefulness for complex workflows
- Requires documentation and awareness for users

## Labels
- bug
- model-specific
- continuation-protocol
- gpt-4o
- needs-workaround

## Priority
Medium - Has workarounds but affects core functionality

## Related Files
- `/tests/004-task-continuation-protocol/`
- `/src/mindswarm/model_capabilities.py`
- `/prompts/shared/continuation_protocol_structured.md`