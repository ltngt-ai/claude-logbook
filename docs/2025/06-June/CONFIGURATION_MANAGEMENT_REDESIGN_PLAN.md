# Configuration Management Redesign Plan

## Date: 2025-06-07

## Context

User requested a plan for user-friendly configuration management following best practices for server applications, with proper API key handling and support for multiple service providers.

## Current State Analysis

### Problems Identified:
1. **Fragmented Configuration**: Different components use different config approaches
2. **No Type Safety**: Most configs are plain dictionaries
3. **Poor Secret Management**: API keys can be stored in plain text YAML
4. **No Environment Support**: No clear dev/staging/prod separation
5. **Limited Validation**: Inconsistent validation across components
6. **No Central Management**: Configuration spread across multiple systems

## Proposed Solution

### Core Architecture

```
Default Values (in code)
    ↓
Schema Defaults
    ↓
Environment-specific Config (dev/staging/prod)
    ↓
Local Config (gitignored)
    ↓
Environment Variables
    ↓
Runtime Overrides
```

### Key Design Decisions

1. **Pydantic Models** for full type safety and validation
2. **SecretStr** for API keys - never stored in config files
3. **Environment-aware** with proper separation
4. **Centralized ConfigurationManager** for single source of truth
5. **Extensible provider system** for easy addition of new services

### Configuration Structure

```python
class MindSwarmConfig(BaseModel):
    environment: Environment
    server: ServerConfig
    logging: LoggingConfig
    
    # Multiple AI providers
    ai_providers: Dict[str, AIProviderConfig] = {
        "openrouter": OpenRouterConfig(),
        "openai": OpenAIConfig(),
        "anthropic": AnthropicConfig(),
    }
    
    # External services
    github: Optional[GitHubConfig]
    database: Optional[DatabaseConfig]
    
    # Feature flags
    features: Dict[str, bool]
```

### Security Features

1. **API Keys**: Only via environment variables using `SecretStr`
2. **Validation**: Required keys checked for production
3. **No Secrets in Code**: Config files contain structure only
4. **.env.example**: Template without real keys
5. **local.yaml**: Gitignored for local overrides

### Usage Benefits

1. **Type Safety**: 
   ```python
   config.server.port  # IDE knows this is an int
   ```

2. **Secure Access**:
   ```python
   api_key = config.ai_providers['openrouter'].api_key.get_secret_value()
   ```

3. **Environment Aware**:
   ```python
   if config.is_production:
       # Production-specific behavior
   ```

4. **Feature Flags**:
   ```python
   if config.features.get('async_agents'):
       # Feature-specific code
   ```

## Implementation Plan

### Phase 1: Core Infrastructure
- Create Pydantic models
- Implement ConfigurationManager
- Add validation and error handling

### Phase 2: Migration
- Update existing code gradually
- Add backwards compatibility
- Migrate all config access points

### Phase 3: Enhanced Features
- Hot-reloading support
- Secret rotation
- Configuration versioning

### Phase 4: Documentation
- Auto-generated docs from Pydantic models
- Migration guides
- Configuration examples

## Technical Advantages

1. **Full Type Safety**: Pydantic provides complete typing
2. **Automatic Validation**: Invalid configs fail fast with clear errors
3. **Self-Documenting**: Models serve as documentation
4. **IDE Support**: Auto-completion and type checking
5. **Secure by Default**: Secrets require explicit handling
6. **Extensible**: Easy to add new providers/services
7. **Environment Support**: Proper dev/staging/prod workflows

## Example Directory Structure

```
config/
├── schemas/                    # Validation schemas
├── environments/              # Environment configs
│   ├── development.yaml
│   ├── staging.yaml
│   └── production.yaml
├── templates/                 # Examples
│   ├── .env.example
│   └── local.yaml.example
├── config.yaml               # Base config (no secrets!)
└── local.yaml               # Local overrides (gitignored)
```

This design provides a modern, secure, and maintainable configuration system that can scale with the project's needs while following industry best practices.