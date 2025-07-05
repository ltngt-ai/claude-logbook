# 2025-06-26: PathManager Singleton Removal

## Issue
After removing the singleton pattern from PathManager and implementing worktree support, several parts of the codebase were still trying to use `PathManager.get_instance()`, causing server startup errors.

## Solution
Replaced all `PathManager.get_instance()` calls with the new constants from `mindswarm.utils.constants`:

### Files Updated

1. **agent_manager.py**
   - Replaced `PathManager.get_instance().project_path / "templates"` with `TEMPLATES_DIR`

2. **yaml_tool_mixin.py**
   - Replaced `PathManager.get_instance().app_path / 'config' / 'tools'` with `TOOLS_CONFIG_DIR`

3. **knowledge_pack_service.py**
   - Replaced `PathManager.get_instance().app_path / "knowledge_packs" / "core"` with `MINDSWARM_ROOT / "knowledge_packs" / "core"`

4. **prompt_system.py**
   - Replaced all project path references with global constants (CONFIG_DIR, AGENTS_CONFIG_DIR, etc.)
   - Removed project-specific prompt logic (deferred to issue #308)
   - Now uses only global config paths

5. **prompt_system_unified.py**
   - Replaced template and config directory references with constants
   - Removed PathManager imports

## Key Decisions

1. **No Project-Specific Prompts Yet**: Since we have GitHub issue #308 for project-level prompt/config overrides, we simplified the prompt system to only use global constants.

2. **Global Constants**: All paths now reference the Mind-Swarm installation directory constants rather than trying to access project-specific paths.

3. **Clean Break**: No backward compatibility - removed all references to the old singleton pattern.

## Result

- Server starts successfully without PathManager.get_instance() errors
- All tools load properly
- System agents deploy correctly
- Cleaner architecture without singleton anti-pattern

## Next Steps

When implementing project-level overrides (issue #308), we'll need to:
- Pass project context to PromptSystem
- Add PathManager instance to prompt resolution
- Support project-specific template/prompt directories

---

*Project: mindswarm-core-private*  
*Clean architecture, clean implementation*