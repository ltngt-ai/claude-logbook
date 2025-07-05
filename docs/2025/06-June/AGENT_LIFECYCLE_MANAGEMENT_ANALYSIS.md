# Agent Instance Management and Lifecycle Analysis

## Context
Analysis of Mind-Swarm requirements for agent instance management and lifecycle based on existing FQN system, hierarchical contexts, and task orchestration architecture.

## Key Findings from Codebase Analysis

### Existing Architecture Components

1. **FQN System** (`agents/fqn.py`)
   - Hierarchical naming: `project.task.agent.instance`
   - Instance tracking and allocation
   - Agent resolution and fuzzy matching
   - Instance number management

2. **Agent Manager** (`services/agents/agent_manager.py`)
   - Current async agent sessions with states
   - Background task processing
   - Sleep/wake cycles and event handling
   - Basic lifecycle management

3. **State Hierarchy** (`state/hierarchy.py`)
   - Three-tier state management: Server → Project → Task
   - Hierarchical context building
   - State persistence and retrieval

4. **Task Orchestrator** (`task/orchestrator.py`)
   - Task lifecycle management
   - Agent team assignment
   - Task execution coordination

5. **Communication System** (`core/communication/mailbox.py`)
   - Agent-to-agent messaging
   - Priority-based message handling
   - Agent aliases and resolution

### Current Agent States
```python
class AgentState(Enum):
    IDLE = "idle"
    ACTIVE = "active"
    SLEEPING = "sleeping"
    WAITING = "waiting"
    STOPPED = "stopped"
```

## Analysis and Recommendations

### 1. Agent Lifecycle States and Transitions

**Recommended Enhanced States:**
```python
class AgentLifecycleState(Enum):
    # Creation states
    INITIALIZING = "initializing"
    READY = "ready"
    
    # Execution states
    IDLE = "idle"
    ACTIVE = "active"
    BUSY = "busy"
    
    # Coordination states
    WAITING = "waiting"
    BLOCKED = "blocked"
    
    # Suspension states
    SLEEPING = "sleeping"
    HIBERNATING = "hibernating"
    
    # Termination states
    STOPPING = "stopping"
    STOPPED = "stopped"
    FAILED = "failed"
    
    # Resource management
    SUSPENDED = "suspended"
    EVICTED = "evicted"
```

**State Transition Rules:**
- INITIALIZING → READY → IDLE (startup)
- IDLE ↔ ACTIVE ↔ BUSY (normal operation)
- Any state → SLEEPING/HIBERNATING (suspension)
- Any state → STOPPING → STOPPED (graceful shutdown)
- Any state → FAILED (error conditions)

### 2. Agent Instance Operations

**Core Operations Needed:**
- `spawn_agent(fqn, config)` - Create new instance
- `clone_agent(source_fqn, target_fqn)` - Clone existing agent
- `migrate_agent(fqn, target_resource)` - Move agent
- `suspend_agent(fqn, reason)` - Suspend for resource management
- `resume_agent(fqn)` - Resume suspended agent
- `terminate_agent(fqn, graceful=True)` - Stop instance
- `evict_agent(fqn)` - Force removal for resource reclaim

### 3. Agent Instance Tracking and Allocation

**Enhanced Instance Manager Design:**
```python
@dataclass
class AgentInstanceInfo:
    fqn: AgentFQN
    state: AgentLifecycleState
    created_at: datetime
    last_activity: datetime
    resource_allocation: ResourceAllocation
    health_status: HealthStatus
    parent_fqn: Optional[AgentFQN]  # For clones
    children_fqns: Set[AgentFQN]    # Child instances
    metadata: Dict[str, Any]

class ResourceAllocation:
    memory_mb: int
    cpu_cores: float
    storage_mb: int
    network_bandwidth: int
    gpu_allocation: Optional[str]

class HealthStatus:
    is_healthy: bool
    last_heartbeat: datetime
    error_count: int
    consecutive_failures: int
    health_check_failures: int
```

### 4. Integration Points with Existing Systems

**FQN Integration:**
- Extend `AgentInstanceManager` with enhanced resource tracking
- Add resource-aware instance allocation
- Implement instance families for related agents

**State Hierarchy Integration:**
- Store agent instance state at TASK level
- Use PROJECT level for resource pools
- Use SERVER level for global policies

**Task Orchestrator Integration:**
- Agent lifecycle hooks in task execution
- Automatic scaling based on task load
- Resource optimization during task transitions

**Communication Integration:**
- Health monitoring via heartbeat messages
- Resource coordination messages
- Agent discovery and routing updates

### 5. Multi-Agent Coordination Features

**Key Features Needed:**

1. **Agent Families/Groups:**
   ```python
   class AgentFamily:
       family_id: str
       parent_fqn: AgentFQN
       children: Set[AgentFQN]
       coordination_policy: CoordinationPolicy
       resource_budget: ResourceBudget
   ```

2. **Resource Pools:**
   ```python
   class ResourcePool:
       pool_id: str
       total_resources: ResourceAllocation
       allocated_resources: ResourceAllocation
       available_resources: ResourceAllocation
       agents: Set[AgentFQN]
       policies: ResourcePolicy
   ```

3. **Coordination Policies:**
   - Load balancing strategies
   - Failover mechanisms
   - Scaling triggers and thresholds
   - Resource sharing rules

4. **Health Monitoring:**
   - Heartbeat tracking
   - Performance metrics collection
   - Failure detection and recovery
   - Circuit breaker patterns

## Implementation Priority

1. **Phase 1: Enhanced States and Transitions**
   - Extend AgentState enum
   - Implement state transition validation
   - Add state persistence to hierarchy

2. **Phase 2: Resource Management**
   - ResourceAllocation and monitoring
   - Instance allocation optimization
   - Resource pool management

3. **Phase 3: Advanced Coordination**
   - Agent families and groups
   - Multi-instance coordination
   - Advanced scheduling

4. **Phase 4: Monitoring and Recovery**
   - Health monitoring system
   - Automatic recovery mechanisms
   - Performance optimization

## Next Steps
1. Implement enhanced lifecycle states
2. Design resource allocation system
3. Extend FQN system for resource awareness
4. Build coordination primitives
5. Implement health monitoring