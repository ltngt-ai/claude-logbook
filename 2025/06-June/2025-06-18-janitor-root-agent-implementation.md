# 2025-06-18: Janitor Bug Fixes and Root Agent Implementation

## Summary
Fixed critical bugs with the Janitor Agent and implemented a Root Agent to solve the system bootstrap problem. This creates a clean initialization chain similar to Unix's init process.

## Problems Solved

### 1. Janitor Agent Duplicate Logs
- **Issue**: Two narrative log files were being created for each janitor agent
- **Root Cause**: Agent registry was converting "janitor_agent" to "JANITOR" (uppercase), but deployment was using "janitor_agent" (lowercase)
- **Fix**: Changed JanitorDeploymentService to use `JANITOR_AGENT_ID = "JANITOR"`

### 2. Schedule Wakeup Context Bug
- **Issue**: All scheduling tools were failing with "No context available" error
- **Root Cause**: Tools were looking for `_context` in kwargs instead of using `self.context` from BaseTool
- **Fix**: Updated all three scheduling tools to use `self.context`

### 3. Bootstrap Problem (Who Sends First Mail?)
- **Issue**: Infrastructure agents were receiving mail from non-existent "system.system"
- **Solution**: Created Root Agent (PID 1) that:
  - Starts without needing an initial mail (self-bootstrapping)
  - Sends bootstrap mails to infrastructure agents
  - Acts as /dev/null for replies (prevents infinite politeness loops)
  - Serves as the source for all scheduled wake-ups (system interrupts)

## Architecture Changes

### Root Agent Design
```yaml
# config/agents/root.yaml
agent_type: root_agent
display_name: Root Agent
icon: "🌱"
tool_sets: ["communication"]  # Only needs to check mail
shared_components:
  - agent_identity  # Minimal components
metadata:
  is_infrastructure: true
  protected: true
  is_root: true
```

### Bootstrap Chain
```
ROOT (PID 1) → Infrastructure Agents → System Operations
```

### Wake-up Architecture
- All scheduled wake-ups now come from ROOT
- Prevents reply loops since ROOT doesn't reply
- Tracks `scheduled_by` to know who requested the wake-up
- Conceptually clean: system interrupts come from "outside"

## Code Changes

### Files Modified
1. `src/mindswarm/services/agents/agent.py` - Fixed agent_id property
2. `src/mindswarm/services/janitor_deployment.py` - Use ROOT as sender
3. `src/mindswarm/tools/scheduling/*.py` - Fixed context access, ROOT as sender
4. `src/mindswarm/server/main.py` - Deploy ROOT before janitor
5. `config/agents/janitor.yaml` - Cleaned up prompts
6. `tests/unit/tools/scheduling/test_scheduling_tools.py` - Updated tests

### Files Created
1. `config/agents/root.yaml` - Root agent configuration
2. `src/mindswarm/services/root_deployment.py` - Root deployment service

## Key Learnings

### Agent Prompt Design
- Agents shouldn't know about narrative logs (they're automatic)
- Don't tell agents about future improvements ("for now")
- Remove impossible tasks from prompts
- Keep infrastructure agents minimal and focused

### Bootstrap Philosophy
- Every system needs a beginning (PID 1)
- ROOT serves multiple purposes efficiently:
  - Bootstrap source
  - /dev/null for acknowledgments
  - System clock/scheduler
- Reuse existing components creatively

## Testing Results
- Fixed 7 failing scheduling tool tests
- Updated to use proper ToolContext pattern
- All tests now passing

## PR Details
- PR #184: https://github.com/ltngt-ai/mindswarm-core-private/pull/184
- Successfully merged to main

## Next Steps
Looking at open GitHub issues, potential areas to explore:
- Issue #180: Implement Agent-First Authentication System
- Issue #178: Implement MCP stdio/SSE transport
- Issue #175: Audit documentation for agent-first architecture
- Issue #167: Update xfailed tests to use new ToolContext pattern

The Root Agent implementation demonstrates the power of the agent-first architecture - even system initialization is handled by an agent, maintaining consistency throughout the system.