# Project Council Architecture - January 2025

## Major Architecture Decision: Project Council System

Today we made a significant architectural decision to replace the single `project_manager` agent with a council of specialized agents. This is a natural evolution that better embodies our agent-first principles.

## The Council Members

1. **Project Speaker** - The spokesagent
   - External interface for all user communication
   - Receives kick-off mail from project_creator
   - Translates between user language and internal protocols
   - Can request reports from Scribe for user updates

2. **Project Minister** - Strategic oversight
   - Monitors project health and alignment with goals
   - Has final say on big decisions requiring consensus
   - Guards against changes that might break the project
   - Natural escalation point for complex issues

3. **Project Librarian** - Knowledge curator
   - Maintains current, accurate knowledge for working teams
   - Integrates with future mind-swarm librarian system
   - Focuses on "what is true now"
   - No historical baggage, just current facts

4. **Project Foreman** - Task manager
   - Handles full task lifecycle (create, assign, review, complete)
   - Coordinates with teams on execution
   - Ensures work quality through reviews

5. **Project Scribe** - The historian
   - Maintains project logbook with full history
   - Records council decisions and discussions
   - Captures the "why" and "how we got here"
   - Produces reports for Speaker to share with users

## Key Design Insights

### Scribe vs Librarian Distinction
This was a crucial refinement. The Librarian maintains current truth for teams doing work, while the Scribe maintains historical context and decision rationale. This solves the eternal documentation problem of mixing historical context with current state.

### Implementation Approach
The implementation is beautifully agent-first:
- Minimal core code change: project_creator spawns council instead of single agent
- Everything else lives in runtime (templates, knowledge packs, protocols)
- Inter-council communication starts with direct agent-to-agent email
- We can observe patterns and optimize later

### Decision-Making Protocol
- Small decisions: Individual agents act autonomously
- Medium decisions: Relevant agents coordinate
- Big decisions: Full council consensus with Minister having final say

## Benefits
1. **Better scalability** - Work naturally distributes across specialized agents
2. **No single point of failure** - Council can function even if one member is busy
3. **Specialized expertise** - Each agent can focus on their domain
4. **More agent-first** - Replaces an overloaded single agent with a natural team

## Issues Created
- Core: https://github.com/ltngt-ai/mind-swarm-core/issues/30
- Runtime: https://github.com/ltngt-ai/mind-swarm-runtime/issues/54

## Implementation Notes
- Start with direct agent-to-agent email for council communication
- Monitor email patterns to see if dedicated channels are needed
- Council protocols will be defined in starter knowledge packs
- Most iteration can happen purely in runtime after core bootstrap

This is exactly the kind of evolution we want to see - moving from monolithic agents to specialized teams that collaborate naturally through our messaging infrastructure.