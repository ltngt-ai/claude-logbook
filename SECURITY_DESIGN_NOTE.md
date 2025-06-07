# Security Design Note - June 7, 2025

## Filesystem Tool Separation

User correctly pointed out that `readonly_filesystem/` and `write_filesystem/` tools should NOT be merged. This is an intentional security design decision.

### Rationale

1. **Principle of Least Privilege**: By keeping read and write operations in separate modules, it's easier to grant agents only the permissions they need.

2. **Accidental Permission Escalation**: Merging them could make it too easy to accidentally grant write privileges when only read access is intended.

3. **Clear Intent**: The separation makes it immediately obvious which tools can modify the filesystem vs just read from it.

4. **Audit Trail**: Easier to audit which agents have been given write capabilities by checking which tool modules they import.

### Updated Plan

The reorganization plan has been updated to maintain this separation:

```
agents/tools/
├── readonly_filesystem/    # Read-only operations
│   ├── read_file.py
│   ├── list_directory.py
│   ├── search_files.py
│   ├── find_pattern.py
│   └── get_file_content.py
├── write_filesystem/       # Write operations (privileged)
│   ├── write_file.py
│   ├── create_directory.py
│   └── delete_file.py
```

This maintains the security boundary while still consolidating tools in a more organized structure.

## Lesson Learned

When refactoring, it's crucial to understand the reasoning behind existing design decisions before changing them. What might look like poor organization could be an intentional security measure.