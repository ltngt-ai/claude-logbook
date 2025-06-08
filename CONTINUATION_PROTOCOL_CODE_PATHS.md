# Continuation Protocol - Code Path Analysis

## Date: 2025-06-08

### Understanding the Two Code Paths

The continuation protocol test (004) is specifically designed to test both code paths:

1. **Structured Output Path (GPT-4o)**
   - Model capability: `"structured_output": True`
   - Uses JSON Schema validation via `response_format`
   - Continuation is a direct field in the response: `{"response": "...", "continuation": {"status": "CONTINUE", "reason": "..."}}`
   - Detection code: Checks `result.get("continuation", {})` directly

2. **Non-Structured Output Path (Claude-2.1)**
   - Model capability: `"structured_output": False`
   - Returns plain text with embedded JSON
   - Continuation JSON is appended to the response text
   - Detection code: Uses regex to extract `{"continuation": {...}}` from the text

### Code Flow Analysis

#### 1. Structured Output Setup (GPT-4o)
```python
# In stateless_session_manager.py:800-814
elif self._should_use_structured_continuation(agent):
    # Check if model supports structured output
    if supports_structured_output(agent.config.model_name):
        kwargs['response_format'] = self._get_continuation_schema()
        logger.info("Enabling structured output for continuation protocol")
```

The continuation schema is loaded from `/home/deano/projects/AIWhisperer/config/schemas/continuation_schema.json`:
```json
{
  "type": "object",
  "required": ["response", "continuation"],
  "properties": {
    "response": {"type": "string"},
    "continuation": {
      "type": "object",
      "required": ["status", "reason"],
      "properties": {
        "status": {"enum": ["CONTINUE", "TERMINATE"]},
        "reason": {"type": "string"}
      }
    }
  }
}
```

#### 2. Continuation Detection (agent_manager.py:601-636)
```python
# Structured output detection
if isinstance(result, dict) and "continuation" in result:
    continuation = result.get("continuation", {})
    if isinstance(continuation, dict) and continuation.get("status") == "CONTINUE":
        should_continue = True
        continuation_reason = continuation.get("reason", "Continuing task")
        logger.info(f"🔄 Structured continuation detected: {continuation_reason}")

# Non-structured output detection
else:
    final_content = channel_response.get("final", "")
    if final_content:
        continuation_match = re.search(r'\{"continuation":\s*\{[^}]+\}\}', final_content)
        if continuation_match:
            continuation_json = json.loads(continuation_match.group())
            continuation = continuation_json.get("continuation", {})
            if continuation.get("status") == "CONTINUE":
                should_continue = True
                continuation_reason = continuation.get("reason", "Continuing task")
                logger.info(f"🔄 Non-structured continuation detected: {continuation_reason}")
```

### Test Results Explained

- **Claude-2.1** (non-structured): ✅ PASSES all 3 phases
  - Successfully embeds continuation JSON in text responses
  - Regex extraction works correctly
  - Continues through all phases

- **GPT-4o** (structured): ❌ FAILS at Phase 3
  - Successfully uses structured output format
  - Continues from Phase 1 to Phase 2
  - Does NOT continue from Phase 2 to Phase 3
  - This suggests the model is returning `{"status": "TERMINATE"}` after Phase 2

### Root Cause Hypothesis

The issue is not with the code paths themselves - both are working correctly. The problem is that GPT-4o is deciding to terminate after Phase 2 rather than continue to Phase 3. This could be due to:

1. **Model interpretation differences**: GPT-4o may interpret the task as complete after creating the summary
2. **Prompt clarity**: The structured continuation prompt may need to be more explicit about multi-phase tasks
3. **Context handling**: GPT-4o might be losing track of the overall task requirements

### Recommendations

1. Add more explicit phase tracking in the continuation schema
2. Include clearer instructions about incomplete phases in the structured continuation prompt
3. Consider adding a "phases_completed" field to help models track progress
4. Test with more explicit prompts that enumerate all required phases upfront