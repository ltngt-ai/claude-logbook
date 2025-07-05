# Mind-Swarm Code Hygiene Cleanup Plan
Date: 2025-01-16

## Overview
This plan tracks the systematic cleanup of Mind-Swarm codebase to improve code hygiene and complete the agent-first architecture migration.

## Phase 1: Remove System Assistant (1-2 days)
**Priority: HIGH - Blocks agent-first architecture**

### Tasks:
1. **Core Removal** (Issue #155)
   - Remove `src/mindswarm/server/system_init.py`
   - Remove initialization from `main.py`
   - Remove related tests
   - Test that server starts without System Assistant

2. **Frontend Removal** 
   - UI: Remove from mindswarm-ui-private (Issue #20)
   - CLI: Remove from mindswarm-cli-private (Issue #4)
   - Ensure no broken references

## Phase 2: Fix Critical Bugs (2-3 days)
**Priority: HIGH - Current functionality broken**

1. **Agent Model Configuration** (Issue #142)
   - Fix agents to use their configured models from `agents.yaml`
   - Keep GPT-3.5 workarounds but ensure they're properly gated by model_capability quirks
   - Verify capability detection works correctly for each model
   - Test each agent type uses correct model

2. **UI Agent Error Protocol** (Issue #151)
   - Implement JSON error response format
   - Update UI agent prompt with error examples
   - Test error handling in frontend

## Phase 3: Mail System Cleanup (2-3 days)
**Priority: MEDIUM - Architectural consistency**

1. **Remove Message Types** (Issue #141)
   - Remove all `type` fields from mail bodies
   - Use subject as single source of truth
   - Update agent prompts to focus on subjects

2. **Email Validation** (Issue #146)
   - Implement proper RFC 2822 validation
   - Replace simple '@' checks
   - Add validation tests

3. **Update Tests** (Issue #147)
   - Fix skipped tests for new email format
   - Ensure all tests use proper addresses

## Phase 4: Complete Architecture Cleanup (3-4 days)
**Priority: HIGH - Long-term maintainability**

1. **Remove Non-Agent Services** (Issue #63)
   - Remove `assistant_service.py`
   - Remove `ResponseEnhancerService`
   - Ensure everything goes through agents

2. **Code Organization**
   - Clean up any remaining backward compatibility code
   - Remove unused imports and dead code
   - Update codemaps

## Issue Tracking
- [ ] Issue #155: Remove System Assistant from core
- [ ] Issue ui#20: Remove System Assistant from UI
- [ ] Issue cli#4: Remove System Assistant from CLI
- [ ] Issue #142: Fix agents using configured models
- [ ] Issue #151: UI Agent error protocol for JSON responses
- [ ] Issue #141: Remove message types from mail system
- [ ] Issue #146: Fix email validation to use proper RFC 2822
- [ ] Issue #147: Update tests for RFC 2822 email format
- [ ] Issue #63: Complete architecture drift cleanup

## Starting Point
Beginning with Phase 1: System Assistant Removal on 2025-01-16.