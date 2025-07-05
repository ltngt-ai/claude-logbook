# Wednesday, June 25, 2025

## AI Service Provider Extension (Issue #281)

### Investigation Summary
Investigated extending Mind-Swarm's AI service providers beyond OpenRouter. Key findings:

1. **Current Architecture**:
   - Single provider (OpenRouter) hardcoded in `ai_loop_factory.py`
   - Model capabilities statically defined in `model_capabilities.py`
   - Basic reasoning token support already implemented
   - No cost tracking or context size management

2. **Research Repository Findings**:
   - Complete Claude Max implementation using browser-based auth
   - No API key required - uses Claude.ai subscription cookies
   - Proven integration pattern from Manus.im research

3. **Implementation Plan Created**:
   - Dynamic model registry for runtime capability discovery
   - Token usage and cost tracking system
   - Context size management to prevent overflows
   - Enhanced reasoning/thinking support
   - Claude Max as new provider option
   - Provider factory pattern for extensibility

### Sub-Issues Created
- #282: Dynamic Model Registry
- #283: Token Usage and Cost Tracking
- #284: Context Size Management
- #285: Reasoning/Thinking Features
- #286: Claude Max Provider
- #287: Provider Factory Pattern

### Architecture Decisions
1. **Provider Factory Pattern**: Clean abstraction for multiple providers
2. **Unified Model Registry**: Single source of truth for capabilities
3. **Agent-First Approach**: All provider interactions through agent tools

### Configuration Update Needed
As Deano noted, we need to update configs to specify both provider AND model:
```yaml
ai_config:
  provider: "claude_max"  # or "openrouter"
  model: "claude-3.5-sonnet"
  # ... other settings
```

This allows per-agent provider selection and fallback chains.

### Next Steps
Starting implementation with the Provider Factory Pattern (#287) as it's the foundation for other features.

## Implementation Progress

### Completed Components

1. **Provider Factory Pattern (#287)** ✅
   - Created AIProviderFactory for managing multiple AI service providers
   - Updated AIConfig to include provider field and provider-specific settings
   - Refactored AILoopFactory to use the new provider factory
   - Added provider/model info to agent narrative logs
   - All tests passing

2. **Dynamic Model Registry (#282)** ✅
   - Created ModelRegistry for runtime model capability discovery
   - Implemented model info fetching from providers with caching (1-hour TTL)
   - Extended model_capabilities.py with new helper functions:
     - get_model_context_length()
     - get_model_pricing()
     - supports_reasoning()
     - supports_vision()
   - Integrated with existing get_model_capabilities() function
   - Comprehensive test coverage
   - Note: "No provider registered" warnings are expected until providers register themselves

### Provider/Model in Narrative Logs
Added provider and model information to agent narrative logs per Deano's request:
```
🤖 MODEL CONFIGURATION:
──────────────────────────────────────────────────
Provider: openrouter
Model: openai/gpt-4.1-mini
Temperature: 0.7
Max Tokens: 1000
──────────────────────────────────────────────────
```

### Next Implementation Target
Moving on to Token Usage and Cost Tracking (#283) to leverage the pricing information from the model registry.