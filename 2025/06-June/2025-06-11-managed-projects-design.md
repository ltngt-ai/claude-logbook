# Managed Projects and Tasks Design

## Overview

This design implements GitHub issue #47 - creating AI-managed projects where intelligent agents autonomously manage the project lifecycle. The system already has foundational pieces in place (managed protocol, mixins, message types), but they need to be connected and activated.

## Architecture

### Hierarchy
```
User
  ↓ (mailbox)
Project Manager Agent
  ↓ (managed protocol)
Task Manager Agents  
  ↓ (managed protocol)
Worker Agents
```

### Key Components

1. **Managed Project Creation**
   - Add `is_managed` flag to project creation
   - Auto-spawn project_manager agent when `is_managed=true`
   - Project manager receives initial configuration and goals

2. **Agent Templates**
   - `project_manager` - Strategic planning, monitoring GitHub, prioritization
   - `task_manager` - Tactical execution, worker coordination
   - Standard worker agents with managed protocol support

3. **Communication Flow**
   - Uses existing mailbox system with managed protocol overlay
   - Structured messages (already defined in managed_messages.py)
   - Channel-based broadcasting for efficiency

## Implementation Plan

### Phase 1: Enable Managed Projects

1. **Update Project Model**
   ```python
   class Project(BaseModel):
       is_managed: bool = Field(default=False, description="AI-managed project")
       project_manager_agent_id: Optional[str] = Field(None, description="Project manager agent ID")
   ```

2. **Update Project Creation**
   ```python
   def create_project(self, project_data: ProjectCreate) -> Project:
       # ... existing code ...
       
       if project_data.is_managed:
           # Auto-create project manager agent
           pm_agent = self._create_project_manager(project)
           project.project_manager_agent_id = pm_agent.instance_id
   ```

3. **Project Manager Agent Template**
   ```yaml
   type: project_manager
   name: "Project Manager"
   model: "gpt-4o"  # Strong model for strategic decisions
   system_prompt: |
     You are a Project Manager agent for a Mind-Swarm managed project.
     
     Your responsibilities:
     - Monitor project goals and GitHub issues
     - Create and prioritize tasks
     - Coordinate with Task Managers
     - Report progress to users
     - Make strategic decisions about project direction
     
     You have access to:
     - GitHub integration tools
     - Task creation/management tools  
     - Communication via managed protocol
     - Project state and history
   
   tools:
     - github_list_issues
     - github_get_issue
     - create_task
     - list_tasks
     - send_mail
     - query_project_state
   ```

### Phase 2: Task Manager Implementation

1. **Managed Task Creation**
   - When PM creates a managed task, auto-spawn task_manager agent
   - Task manager receives task details and requirements
   - TM decides which workers are needed

2. **Task Manager Agent Template**
   ```yaml
   type: task_manager
   name: "Task Manager"
   model: "gpt-4o-mini"  # Lighter model for tactical work
   system_prompt: |
     You are a Task Manager agent responsible for executing a specific task.
     
     Your responsibilities:
     - Understand task requirements
     - Find and coordinate worker agents
     - Monitor progress and quality
     - Report back to Project Manager
     - Handle blockers and issues
     
     Use the managed protocol for all communications.
   
   tools:
     - list_agents
     - spawn_agent
     - send_mail
     - get_task_details
   ```

### Phase 3: Self-Correction Mechanisms

1. **Peer Review System**
   - Multiple task managers can review each other's work
   - Project manager reviews critical decisions
   - Workers can report issues up the chain

2. **Feedback Loops**
   ```
   Worker → Task Manager: "I'm blocked on X"
   Task Manager → Project Manager: "Need help with X"
   Project Manager → User: "Requesting guidance on X"
   ```

3. **Quality Gates**
   - Task completion requires TM approval
   - Major decisions require PM confirmation
   - User can always override

## Testing Strategy

### 1. Unit Tests
- Test managed protocol message passing
- Test agent spawn on project/task creation
- Test role-based message routing

### 2. Integration Tests
- Create managed project → verify PM agent created
- PM creates task → verify TM agent created
- Full flow: User request → PM → TM → Worker → Result

### 3. Behavioral Tests
- Give PM a GitHub issue → verify it creates appropriate tasks
- Introduce a blocker → verify escalation chain works
- Test self-correction when worker fails

### 4. Observability
- Enhanced narrative logging for managed agents
- Metrics on task completion rates
- Decision audit trail

## Implementation Challenges

1. **Prompt Engineering**
   - Iterative refinement of manager prompts
   - Clear role boundaries
   - Avoiding infinite loops or deadlocks

2. **Resource Management**
   - Preventing agent proliferation
   - Cleanup of completed task managers
   - Cost control with AI calls

3. **User Control**
   - Override mechanisms
   - Visibility into AI decisions
   - Manual intervention points

## Next Steps

1. Start with minimal implementation:
   - Add `is_managed` to project creation
   - Create basic project_manager template
   - Test with simple scenarios

2. Iterate on prompts based on behavior
3. Add more sophisticated tools and capabilities
4. Implement self-correction mechanisms

## Success Metrics

- Projects can run autonomously for extended periods
- Correct task decomposition from GitHub issues  
- Appropriate escalation of blockers
- User satisfaction with AI management
- Reduction in manual project management overhead