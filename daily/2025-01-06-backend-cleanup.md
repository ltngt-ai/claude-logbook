# Backend Cleanup Log - January 6, 2025

## Objective
Streamline mindswarm-core to focus exclusively on async agent workflows over WebSocket. Remove all legacy code, CLI interfaces, batch processing, and implement proper structured logging.

## Analysis Results

### Core Requirements
The backend should only include:
1. **WebSocket API** - For client communication
2. **Async Agent System** - Core agent orchestration
3. **AI Service Integration** - OpenRouter and tool calling
4. **Essential Tools** - Agent communication and lifecycle
5. **Structured Logging** - Replace file proliferation

### Components to Remove
- Entire CLI interface (`cli/`)
- Conversation replay system
- MCP integration (unless actively used)
- AST/code analysis tools
- Legacy model capabilities
- Batch processing components
- Plan/RFC tools (unless needed)

### Logging System Design
Current issues:
- Hundreds of log files with timestamps
- Different log files for each component
- No rotation or cleanup
- Difficult to debug across components

Proposed solution:
- Single structured JSON logger
- Log to stdout/stderr for container compatibility
- Optional file output with rotation
- Correlation IDs for request tracking
- Log levels: ERROR, WARNING, INFO, DEBUG

## Implementation Plan

### Phase 1: Set up new logging system
1. Create structured logging module
2. Replace all file-based logging
3. Add correlation ID tracking
4. Configure log rotation if file output needed

### Phase 2: Remove legacy code
1. Delete CLI directory
2. Remove conversation replay
3. Clean up unused tools
4. Remove model capabilities system
5. Delete batch processing code

### Phase 3: Refactor core
1. Simplify configuration
2. Consolidate agent management
3. Clean up dependencies
4. Update tests for new structure

---
Starting implementation...

## Progress Update - Phase 1 Complete

### Completed Tasks
1. **Created structured logging system** (`src/mindswarm/core/structured_logging.py`)
   - JSON-formatted logs with correlation IDs
   - Rotating file handler support
   - Proper log levels and library configuration
   - No more hundreds of log files!

2. **Removed legacy code** (28 items)
   - Entire CLI interface removed
   - Conversation replay system removed
   - Legacy model capabilities removed
   - AST/code analysis tools removed
   - Plan/RFC tools removed
   - Old test directories cleaned up

3. **Updated imports** (92 files)
   - Changed all `ai_whisperer` imports to `mindswarm`
   - Fixed duplicate imports issue
   - Updated logging imports to use new structured system

4. **Cleaned up tools directory**
   - Removed duplicate tool versions (_old, _original, _optimized, _lazy)
   - Removed unused tool registration files
   - Kept only essential async agent tools

### Current State
The backend is now much slimmer and focused on:
- WebSocket API for client communication
- Async agent orchestration
- AI service integration (OpenRouter)
- Essential tools for agent lifecycle
- Structured JSON logging

### Issues Fixed
- Circular import in structured_logging.py
- Duplicate imports from cleanup script
- Old logging references updated

### Next Steps
1. Test the server startup with new structure
2. Update configuration to use structured logging ✓
3. Remove more legacy code if found during testing
4. Update pyproject.toml dependencies

## Issues Found
- CLI command imports still in api/main.py - need to remove or replace with minimal handler
- The dispatch_command_handler still expects CommandRegistry from CLI
- Need to decide: keep slash commands with minimal implementation or remove entirely