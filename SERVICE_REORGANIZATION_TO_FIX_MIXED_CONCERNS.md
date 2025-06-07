# Service Reorganization to Fix Mixed Concerns

## Date: 2025-06-07

## Problem Identified

User pointed out significant architectural confusion:
- `services/execution` contained context management (agent concern)
- `services/agents` contained AI loop manager
- AI loop factory in execution but manager in agents (circular dependency risk)
- Three separate context systems causing confusion
- Unclear boundaries between services

## Solution Implemented

### 1. Created New Service Structure

```
services/
├── ai_execution/          # All AI loop components together
│   ├── __init__.py
│   ├── ai_loop.py        # Core processing (from execution)
│   ├── ai_config.py      # Configuration (from execution)
│   ├── ai_loop_factory.py # Factory pattern (from execution)
│   ├── ai_loop_manager.py # Manager (from agents - fixes circular dep)
│   └── tool_call_accumulator.py # Tool handling (from execution)
│
├── agents/               # Pure agent lifecycle management
│   ├── agent.py         # Agent class (renamed from stateless.py)
│   ├── config.py        # Agent configuration
│   ├── registry.py      # Agent registry
│   ├── session_manager.py # Session management
│   └── continuation_strategy.py # Continuation handling
│
└── state/               # Dedicated state management
    ├── __init__.py
    └── state_manager.py # From execution/state.py
```

### 2. Consolidated Context Management

```
context/
├── provider.py              # Base interface
├── agent_context.py         # Agent-specific context
├── conversation_manager.py  # From execution/context.py
├── file_context_manager.py  # Renamed from context_manager.py
└── context_item.py         # Context items
```

### 3. Key Changes

1. **Removed `services/execution`** - Split contents appropriately
2. **Fixed circular dependencies** - Used TYPE_CHECKING for AgentConfig in ai_loop_manager
3. **Clear naming** - No more confusion between different context managers
4. **Single responsibility** - Each service has one clear purpose

### 4. Import Updates

Updated all imports throughout the codebase:
- `services.execution.ai_*` → `services.ai_execution.*`
- `services.execution.context` → `context.conversation_manager`
- `services.execution.state` → `services.state.state_manager`
- `services.agents.ai_loop_manager` → `services.ai_execution.ai_loop_manager`

## Benefits Achieved

1. **Clear Separation of Concerns**
   - AI Execution: Core AI processing and management
   - Agents: Agent lifecycle and configuration only
   - State: State persistence and management
   - Context: All context-related functionality

2. **No Circular Dependencies**
   - AI execution can be imported by agents
   - No backward dependencies

3. **Better Maintainability**
   - Each service has single responsibility
   - Clear import paths
   - Obvious where new code should go

4. **Reduced Confusion**
   - No duplicate context managers with similar names
   - Clear file names match their purpose
   - Consistent organization

## Technical Details

- Used TYPE_CHECKING to avoid runtime circular import in ai_loop_manager
- Maintained all existing functionality
- All 441 tests passing
- No breaking changes to external APIs

## Combined with Previous Work

This reorganization built on the previous cleanup:
- Removed all legacy stateful/synchronous components
- Renamed StatelessAgent → Agent
- Removed "stateless" prefix from files
- Made class names match file names

The codebase now has a clean, logical structure with clear boundaries between services.