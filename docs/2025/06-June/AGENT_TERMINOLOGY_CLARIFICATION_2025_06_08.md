# Agent Terminology Clarification
**Date**: June 8, 2025

## Context
While setting up tests, we encountered confusion about agent types, roles, templates, and IDs. This document clarifies the terminology used throughout Mind-Swarm.

## Terminology Mapping

### 1. Agent Type (Template)
- **What it is**: Blueprint for creating agent instances
- **Examples**: `research`, `assistant`, `coordinator`
- **Used in**: CLI commands (`--agent-type`)
- **Maps to**: `role` field in agents.yaml

### 2. Agent ID (Configuration Key)
- **What it is**: Single-letter identifier in configuration
- **Examples**: `a` (assistant), `r` (research), `c` (coordinator)
- **Used in**: agents.yaml as top-level keys
- **Legacy**: From AIWhisperer pattern

### 3. Instance ID
- **What it is**: Unique identifier for a spawned agent
- **Examples**: `roger`, `assistant-1`, `test-research`
- **Used in**: Runtime identification and addressing
- **Purpose**: Distinguish multiple instances of same type

### 4. Role
- **What it is**: The functional role an agent performs
- **Examples**: `assistant`, `research`, `coordinator`
- **Used in**: Internal agent configuration
- **Note**: Often used interchangeably with "type"

## Usage Patterns

### Mind-Swarm Pattern (Recommended)
```bash
mindswarm agent spawn --agent-type research --instance-id roger
```
- Uses template/type + instance ID
- Clear separation of blueprint and instance

### Direct Agent Pattern (Legacy)
```bash
mindswarm agent spawn --agent-id r
```
- Uses single-letter configuration key directly
- Less flexible but more concise

## Configuration Structure

```yaml
agents:
  a:  # Agent ID (single letter)
    name: "Assistant"
    role: "assistant"  # Role/Type
    tool_sets: ["communication", "readonly_filesystem", "write_filesystem"]
```

## CLI to Server Flow

1. **CLI Input**: `--agent-type a` or `--agent-type assistant`
2. **Server Processing**: Looks up by role or agent ID
3. **Template Resolution**: Finds matching configuration
4. **Instance Creation**: Spawns with provided instance ID

## Key Findings

1. **Dual Support**: System supports both patterns for flexibility
2. **Type = Role**: These terms are used interchangeably
3. **Single Letters Work**: Can use `a` instead of `assistant` as type
4. **Tool Inheritance**: Agents get tools from their template

## Recommendations

1. Use descriptive instance IDs (e.g., `research-climate` not `r1`)
2. Prefer full type names in documentation (e.g., `assistant` not `a`)
3. Be consistent within a project about which pattern to use
4. Document which agents have which tools available

This clarification was essential for getting the test framework working correctly.