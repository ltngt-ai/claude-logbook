# 2025-06-30: Storage System Redesign

## Overview

Today I designed a comprehensive storage system redesign for Mind-Swarm to address fragmentation, security gaps, and provide a clean API for tools.

## Context

Deano raised [Issue #16](https://github.com/ltngt-ai/mindswarm-core/issues/16) about cleaning up Mind-Swarm's storage. The current system has:
- Multiple storage implementations (JSON, SQLite, hardcoded)
- Direct file access in tools
- No encryption for sensitive data
- Unclear separation between system/user/project storage
- No audit trails or versioning

## Storage Design

### Architecture

Created a layered architecture:
```
┌─────────────────────────────────────────────────────────────┐
│                     Storage API Layer                        │
│  (Unified interface for all storage operations)             │
├─────────────────────────────────────────────────────────────┤
│                    Storage Service Layer                     │
│  (Business logic, validation, access control)               │
├─────────────────────────────────────────────────────────────┤
│                    Storage Backend Layer                     │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │
│  │    JSON     │ │   SQLite    │ │   Redis     │          │
│  │   Backend   │ │   Backend   │ │  Backend    │          │
│  └─────────────┘ └─────────────┘ └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### Storage Scopes

Introduced four storage scopes with clear access patterns:
- **system://** - Global configuration (admin only)
- **user://** - User-specific data (encrypted secrets)
- **project://** - Project-specific data
- **shared://** - Cross-project resources

### Tool API

Designed a simple API for tools through ToolContext:
```python
# Simple, intuitive methods
await context.storage.put_project_data("namespace", "key", data)
await context.storage.get_user_secret("github_token")
await context.storage.put_state("config", agent_config)
```

### Key Features

1. **Security**
   - Automatic encryption for sensitive data
   - Role-based access control
   - Complete audit trail
   - Path traversal protection

2. **Data Management**
   - Built-in versioning with metadata
   - Schema migrations
   - Backup/restore capabilities
   - Atomic operations

3. **Performance**
   - In-memory caching layer
   - Bulk operations
   - Connection pooling
   - Target: <50ms reads

## Implementation Plan

Designed a 4-phase implementation:
1. **Week 1**: Core infrastructure and backends
2. **Week 2**: Tool integration and API
3. **Week 3**: Data migration
4. **Week 4**: Advanced features

## Documents Created

1. **STORAGE_DESIGN.md** - Complete architectural design
2. **STORAGE_IMPLEMENTATION_GUIDE.md** - Practical code examples
3. **STORAGE_SUMMARY.md** - Executive summary

## Key Decisions

1. **Agent-First**: No direct CRUD endpoints, tools handle storage
2. **Type Safety**: Pydantic models throughout
3. **Pluggable Backends**: Easy to add new storage backends
4. **Backward Compatible**: Parallel run during migration

## Technical Details

### Storage Key Design
```python
class StorageKey:
    def __init__(self, scope: StorageScope, namespace: str, key: str):
        self.scope = scope
        self.namespace = self._sanitize_path(namespace)
        self.key = self._sanitize_path(key)
    
    @property
    def uri(self) -> str:
        return f"{self.scope}://{self.namespace}/{self.key}"
```

### Enhanced JSON Backend
- Atomic writes with temp files
- Separate metadata files
- Automatic versioning
- DateTime serialization

### Access Control Integration
```python
async def get(self, key: StorageKey, model: Type[T], agent_id: str) -> Optional[T]:
    if not await check_storage_permission(agent_id, key, "read"):
        raise PermissionError(f"Agent {agent_id} cannot read {key.uri}")
```

## Migration Strategy

Designed a safe migration approach:
1. New system runs alongside old
2. Tools updated gradually
3. Automated data migration scripts
4. Complete audit of migrated data

## Next Steps

Ready to begin implementation:
1. Create core storage interfaces
2. Implement JSON backend
3. Add storage service to registry
4. Update ToolContext
5. Create first storage tool

## Reflections

This design balances several concerns:
- **Simplicity**: Easy API for tool developers
- **Security**: Built-in protection for sensitive data
- **Performance**: Caching and efficient operations
- **Flexibility**: Pluggable backends for future needs

The agent-first approach means no direct storage endpoints - everything goes through tools, maintaining Mind-Swarm's core principles.

## Code Quality

The design emphasizes:
- Strong typing with TypeVar and Pydantic
- Comprehensive error handling
- Async/await throughout
- Clear separation of concerns

This storage redesign provides the foundation for reliable, secure data persistence across Mind-Swarm.