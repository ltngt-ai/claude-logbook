# Agent Scoping and Terminate Command Fix

## Date: 2025-01-08

## Issues Identified

### 1. Global vs Project Agent Scoping
- **Problem**: Agents are created globally instead of being scoped to projects
- **Current behavior**: Agent IDs are stored directly in `self.sessions` dictionary without project context
- **Expected behavior**: Agents should be scoped to projects (e.g., `project1.agent1` not just `agent1`)

### 2. Terminate Command Parameter Mismatch
- **Problem**: CLI sends `agent_fqn` but server expects `agent_id`
- **Error**: "Invalid params" when trying to terminate agents
- **Root cause**: 
  - CLI client sends: `{"agent_fqn": "test-assistant", "force": false}`
  - Server expects: `{"agent_id": "test-assistant"}`

### 3. FQN System Not Fully Integrated
- **Observation**: The codebase has an FQN (Fully Qualified Name) system in `mindswarm/agents/fqn.py`
- **Format**: `project.task.agent.instance`
- **Status**: Not actively used by the current `AgentManager` implementation

## Solution Approach

### Phase 1: Fix Terminate Command (Quick Fix)
1. Update server to accept both `agent_fqn` and `agent_id` for backward compatibility
2. Extract agent_id from FQN if provided

### Phase 2: Implement Project-Scoped Agents
1. Modify `AgentManager` to store agents with project context
2. Update agent creation to require project_id
3. Implement proper FQN resolution in agent operations
4. Update list_agents to properly filter by project

## Implementation Details

### 1. Fix Terminate Parameter Mismatch

**File**: `/home/deano/projects/mindswarm-core-private/src/mindswarm/server/main.py`

```python
async def _handle_terminate_agent(self, params: dict) -> dict:
    """Terminate an agent."""
    if not self.agent_manager:
        raise ValueError("Agent manager not initialized")
    
    # Support both agent_fqn (new) and agent_id (legacy)
    agent_identifier = params.get("agent_fqn") or params.get("agent_id")
    if not agent_identifier:
        raise ValueError("Either agent_fqn or agent_id must be provided")
    
    # Extract agent_id from FQN if needed
    # FQN format: project.task.agent.instance
    if "." in agent_identifier:
        # For now, just use the last part as agent_id
        agent_id = agent_identifier.split(".")[-1]
    else:
        agent_id = agent_identifier
    
    await self.agent_manager.stop_agent(agent_id)
    return {"success": True}
```

### 2. Implement Project-Scoped Agent Storage

**File**: `/home/deano/projects/mindswarm-core-private/src/mindswarm/services/agents/agent_manager.py`

Add project context to agent sessions:

```python
class AgentManager:
    def __init__(self, config, registry):
        # Change from simple dict to nested structure
        self.sessions = {}  # Will become: {project_id: {agent_id: session}}
        self.global_sessions = {}  # For backward compatibility
        
    async def create_agent_instance(
        self,
        agent_type: Optional[str] = None,
        instance_id: Optional[str] = None,
        agent_id: Optional[str] = None,
        project_id: Optional[str] = None,  # New parameter
        auto_start: bool = True
    ) -> AgentSession:
        # ... existing validation ...
        
        # Determine storage location
        if project_id:
            # Project-scoped agent
            if project_id not in self.sessions:
                self.sessions[project_id] = {}
            
            if session_id in self.sessions[project_id]:
                raise ValueError(f"Agent '{session_id}' already exists in project '{project_id}'")
            
            # Store with project scope
            self.sessions[project_id][session_id] = session
            session.metadata['project_id'] = project_id
            session.metadata['fqn'] = f"{project_id}.{session_id}"
        else:
            # Global agent (backward compatibility)
            if session_id in self.global_sessions:
                raise ValueError(f"Global agent '{session_id}' already exists")
            
            self.global_sessions[session_id] = session
            session.metadata['fqn'] = f"global.{session_id}"
```

### 3. Update Agent Resolution

Add methods to resolve agents by FQN:

```python
def _resolve_agent_session(self, agent_identifier: str) -> Optional[AgentSession]:
    """Resolve agent session by ID or FQN."""
    # Try direct ID lookup in global sessions
    if agent_identifier in self.global_sessions:
        return self.global_sessions[agent_identifier]
    
    # Try FQN resolution
    if "." in agent_identifier:
        parts = agent_identifier.split(".")
        if len(parts) >= 2:
            project_id = parts[0]
            agent_id = parts[-1]
            
            if project_id == "global":
                return self.global_sessions.get(agent_id)
            elif project_id in self.sessions:
                return self.sessions[project_id].get(agent_id)
    
    # Try searching all projects
    for project_id, agents in self.sessions.items():
        if agent_identifier in agents:
            return agents[agent_identifier]
    
    return None

async def stop_agent(self, agent_identifier: str):
    """Stop an agent and clean up resources."""
    session = self._resolve_agent_session(agent_identifier)
    if not session:
        raise ValueError(f"Agent '{agent_identifier}' not found")
    
    # ... rest of existing stop logic ...
```

## Testing Plan

1. Test terminate command with current global agents
2. Test agent creation with project scope
3. Test FQN resolution in all agent operations
4. Test backward compatibility with existing code

## Next Steps

1. Implement Phase 1 (terminate fix) immediately
2. Test with Mind-SwarmSimpleTasks framework
3. Plan Phase 2 implementation based on test results
4. Consider full FQN system integration for future