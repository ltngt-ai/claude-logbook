# 2025-06-11: Narrative Logging Fixes for GitHub Issue #44

## Summary
Fixed multiple issues with the agent narrative logging system that were preventing proper debugging of agent behavior. The narrative log now provides clear visibility into what agents are thinking and doing.

## Issues Fixed

### 1. Tool Names Showing as "Unknown"
- **Problem**: Narrative logger was using incorrect parameter names (e.g., `file_path` instead of `path`)
- **Solution**: Updated all tool parameter references to match actual tool schemas
- **Files**: `agent_narrative_logger.py`

### 2. get_file_content NoneType Error  
- **Problem**: Tool was calling legacy `PathManager.get_instance()` which returned None
- **Root Cause**: This was an artifact from the old "AI Whisperer" era before MindSwarm had project-specific PathManagers
- **Solution**: Updated to use project-specific PathManager from agent context
- **Files**: `get_file_content.py`

### 3. Empty Recipient Validation
- **Problem**: send_mail tool accepted empty recipients, causing invalid addresses like "Frontend.UpdateDocs.agent1.0"
- **Solution**: Added validation to check for empty recipients
- **Files**: `send_mail.py`

### 4. Tool Error Handling
- **Problem**: Narrative log wasn't showing when tools failed
- **Solution**: Check tool results for error dict before marking as success
- **Files**: `ai_loop.py`

### 5. Email Body Logging
- **Key Insight**: "I think our agent is telling us why about the problem, we just aren't listening"
- **Solution**: Added mail body content to narrative log to see what agents are communicating
- **Files**: `agent_narrative_logger.py`

### 6. Parallel Tool Execution
- **Problem**: When agents execute multiple tools at once, the linear narrative log looked confusing
- **Solution**: Added indicators showing when multiple tools are executed in parallel
- **Files**: `ai_loop.py`, `agent_narrative_logger.py`

### 7. read_file Error Handling
- **Problem**: Tool was raising exceptions instead of returning error dicts
- **Solution**: Updated to return proper error dict format
- **Files**: `read_file.py`

## Test Harness Discovery
- Found that test harness was only giving agents 5 seconds to complete tasks
- This wasn't enough time for complex operations like writing code
- Increased timeout to 30 seconds in MindSwarmSimpleTasks
- Agent behavior varies between runs (e.g., using `touch` + `write_file` vs `echo` directly)

## Agent Debugging Insights
- Agents show creativity in solving problems - same task can be solved different ways
- The narrative log is invaluable for understanding agent "thought process"
- Server logs are hard to read (JSON format with lots of tokens)
- Agents can get stuck in loops of empty responses/reasoning
- Important to give agents enough time to complete complex tasks

## Sample Narrative Log Output
```
🛠️ Sending mail to 'user'
   Subject: Task Update: Implementing calculator.py module  
   Body: The search did not find an existing calculator.py file. I will proceed with creating the implementation as requested.
   ✓ Success

🛠️ Running command: echo "def add(a, b):\n    return a + b\n\ndef subtract(a, b):\n    return a - b\n\ndef multiply(a, b):\n    return a * b\n\ndef divide(a, b):\n    try:\n        return a / b\n    except ZeroDivisionError:\n        return 'Error: Division by zero'\n\n" > calculator.py
   ✓ Success
```

## Next Steps
- Consider making server logs more human-friendly
- Investigate why agents sometimes get stuck in empty response loops
- Add more sophisticated task completion detection beyond simple timeouts