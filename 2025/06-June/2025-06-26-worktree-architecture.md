# 2025-06-26: Git Worktree Architecture for Mind-Swarm

## Overview

Today I worked with Deano on implementing Git worktree support for Mind-Swarm based on issue #296. We identified a fundamental problem with the initial implementation and redesigned it to be much more elegant.

## Initial Implementation

First, I implemented the basic worktree functionality:
- Added worktree methods to `GitRepository` class
- Created 5 git tools: `git_worktree_add`, `git_worktree_list`, `git_worktree_remove`, `git_worktree_switch`, `git_worktree_prune`
- Added YAML configurations and a knowledge pack

## The Problem

Deano pointed out a critical issue: the cognitive load was too high. Agents would need to:
1. Understand absolute vs relative paths
2. Manually switch between worktrees
3. Keep track of which worktree they're in
4. Handle complex path resolution

This violated Mind-Swarm's principle of keeping agents simple and focused.

## The Better Solution

We redesigned the architecture around three key principles:

### 1. Manager-Controlled Isolation
- Only Task/Project Managers decide when to use worktrees
- Regular agents never know they're in a worktree
- Removed `git_worktree_switch` - context handles this automatically

### 2. Context Inheritance
```python
# Task Manager creates worktree
worktree = create_worktree("task/123/feature")

# All agents for this task inherit the context
code_writer.context.worktree = worktree

# Agents just work naturally
read_file("src/main.py")  # Automatically reads from worktree
```

### 3. Relative Paths Only
- All file operations use paths relative to project root
- PathManager transparently resolves to correct location
- No more `/home/user/project/.mindswarm/worktrees/xyz/src/file.py`
- Just `src/file.py` - simple and natural

## Architecture Components

### Enhanced PathManager
- `effective_root` property that considers worktree context
- Transparent path resolution
- Security boundaries maintained within worktree

### Task Context System
- TaskContext carries worktree information
- Inherited by all agents working on the task
- Propagates through mail headers and agent spawning

### Migration Strategy
- 5-week plan to move from absolute to relative paths
- Backward compatibility during transition
- Monitoring and metrics for adoption tracking

## Key Insights

1. **Transparency is powerful** - Agents shouldn't need to understand infrastructure
2. **Context inheritance** solves many problems elegantly
3. **Manager agents** are the right place for strategic decisions
4. **Relative paths** reduce cognitive load significantly

## Next Steps

Starting implementation of:
1. PathManager enhancements with worktree awareness
2. Task context inheritance system
3. Migration from absolute to relative paths
4. Update existing file tools

## Technical Decisions

- Worktrees stored in `.mindswarm/worktrees/` (auto-gitignored)
- Worktree names include task ID for easy identification
- Context propagates through UnifiedContext system
- File tools will reject absolute paths after migration

## Reflections

This is a great example of how initial implementations can be improved by stepping back and considering the user experience. The first version was functionally complete but cognitively complex. The redesigned version is both more powerful and simpler to use.

The key was recognizing that worktrees are an implementation detail that most agents shouldn't need to understand. By making them transparent through context, we get all the benefits (isolation, parallel development) without the complexity.

---

*Project: mindswarm-core-private*
*Issue: #296 - Add worktree support for projects*