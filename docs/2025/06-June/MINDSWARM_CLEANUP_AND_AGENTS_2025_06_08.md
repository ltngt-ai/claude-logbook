# Mind-Swarm Cleanup and New Agents
**Date**: June 8, 2025

## Major Accomplishments Today

### 1. Created 4 New Agent Types
Successfully added new specialized agents to Mind-Swarm:
- **Willy the Watcher (w)** - Monitors and reports on activities
- **Patricia the Project Manager (p)** - Manages projects and coordinates task managers
- **Tracey the Task Manager (t)** - Manages individual tasks and coordinates agents
- **Stephen the Software Engineer (s)** - Expert coder and software engineer

Each agent has:
- Simple prompt files (following the pattern of existing agents)
- Appropriate tool sets for their roles
- Unique colors and icons for UI identification
- Single-letter IDs following the existing pattern

### 2. Updated SimpleTasks Framework
- **Test 001**: Now includes all agents with write_filesystem capability (a, p, t, s)
- **Test 002**: Limited to assistant (a) and software engineer (s) for code generation
- Created configuration files for all new agent types
- Fixed research agent to remove write_filesystem (matching main config)

**Test Results**:
- Assistant agent (a) passes both tests successfully
- New agents spawn correctly but don't create files yet (expected with minimal prompts)
- Infrastructure is solid and ready for iterative improvements

### 3. Removed tool_tags in Favor of tool_sets
**Problem**: Both `tool_tags` and `tool_sets` existed but served similar purposes
**Solution**: Standardized on `tool_sets` only

Changes made:
- Removed tool_tags from all agent configurations
- Updated Agent dataclass to remove the field
- Modified tool registry to remove tags parameter
- Simplified code by having a single tool specification mechanism

Benefits:
- tool_sets are more powerful (support inheritance, exclusions)
- Cleaner, less confusing configuration
- Single source of truth for agent tools

### 4. Added CLI Wrapper Script
Created `./mindswarm` script in mindswarm-core-private that:
- Automatically activates the CLI's virtual environment
- Passes all arguments to the mindswarm CLI
- Includes error handling for missing directories/venv
- Makes development much easier

Usage:
```bash
./mindswarm agent list
./mindswarm project create myproject
./mindswarm interactive
```

### 5. Removed Vestigial context_sources Field
**Discovery**: The `context_sources` field was inherited from AIWhisperer but never implemented
**Action**: Completely removed it from the codebase

What was removed:
- All context_sources entries from agents.yaml
- context_sources field from Agent dataclass
- Loading logic that read context_sources
- Old AIWhisperer prompts moved to research folder

## Technical Insights

1. **Agent System Architecture**: 
   - Single-letter IDs map to roles/types
   - Agents are templates that spawn instances
   - Tool availability is controlled by tool_sets

2. **Prompt System Capabilities**:
   - Supports template parameters with `{{{parameter}}}` syntax
   - Hierarchical prompt loading from multiple locations
   - Shared components and feature toggling
   - No need for context_sources - can use template parameters instead

3. **Server Restart Required**: When adding new agent types, the server must be restarted to load the new configurations

## Next Investigation
About to investigate mindswarm.db - suspicion is it's an artifact from before the project system was working properly.

## Personal Reflection
Today was about cleaning up technical debt and establishing a solid foundation for future agent development. The removal of tool_tags and context_sources significantly simplifies the codebase. The new agents provide a framework for specialized behaviors that can be iteratively improved.

The CLI wrapper script is a small but important quality-of-life improvement that will save time during development. These kinds of developer experience improvements compound over time.