# Worklog 003: Telescope.nvim Position Encoding Warning Fix

**Date:** 2025-10-12
**Issue:** LSP position_encoding parameter warning when using 'gd' in Rust files
**Status:** ✅ Resolved

---

## Problem Statement

### Symptom
When invoking `gd` (go to definition) on functions in Rust files, Neovim displayed a warning popup:
```
position_encoding param is required in vim.lsp.util.make_position_params.
Defaulting to position encoding of the first client.
```

### Environment
- **Neovim Version:** 0.11.4 (latest)
- **Telescope Version:** 0.1.x branch (commit `a0bbec21143c7bc5f8bb02e0005fa0b982edc026`)
- **LSP Client:** rust-analyzer (via rustaceanvim plugin)
- **Config Base:** kickstart.nvim with custom plugins

---

## Root Cause Analysis

### Breaking Change in Neovim 0.11.0
Neovim 0.11 introduced a breaking API change requiring explicit `position_encoding` parameter in LSP utility functions:
- `vim.lsp.util.make_position_params()`
- `vim.lsp.util.make_range_params()`
- `vim.lsp.util.make_given_range_params()`
- `vim.lsp.util.symbols_to_items()`

Previously, these functions would silently default to the first attached LSP client's position encoding. In 0.11+, this behavior triggers a deprecation warning.

### Telescope 0.1.x Incompatibility
The Telescope 0.1.x branch was released **before** Neovim 0.11 and does not include compatibility updates. When calling LSP functions internally, it fails to pass the required `position_encoding` parameter.

### Affected Code Path
1. User keymap at `init.lua:1004`:
   ```lua
   map('gd', require('telescope.builtin').lsp_definitions, '[G]oto [D]efinition')
   ```
2. Telescope internally calls `vim.lsp.util.make_position_params()` without the required parameter
3. Neovim warns about defaulting to first client's encoding (rust-analyzer's utf-16)

### Why Multiple Clients Matter
The warning mentions "first client" because:
- Multiple LSP clients can attach to a buffer with different position encodings (utf-8, utf-16, utf-32)
- Without explicit encoding specification, behavior becomes non-deterministic
- Could cause incorrect cursor positioning with mixed LSP clients

---

## Solution

### Primary Fix
**Update Telescope to master branch** which includes Neovim 0.11 compatibility fixes.

**File:** `init.lua:827`

**Change:**
```diff
  'nvim-telescope/telescope.nvim',
  event = 'VimEnter',
- branch = '0.1.x',
+ branch = 'master',
  dependencies = {
```

### Key Commits in Telescope Master
- `a17d611` - fix(lsp): add default position encoding when calling `symbols_to_items()`
- `b4da76b` - fix(lsp): stop using deprecated `client.supports_method` function
- `a4ed825` - fix(lsp): don't return negative values from `item_to_location`

### Implementation Steps
1. Edit `init.lua:827` to change branch specification
2. Run `:Lazy sync telescope.nvim` or restart Neovim
3. Telescope updates from commit `415af52` (0.1.x) to `b4da76b` (master)
4. Test `gd` functionality in Rust file - warning should be gone

---

## Technical Deep Dive

### Position Encoding Explained
LSP clients negotiate character position encoding with servers:
- **utf-8**: 1 byte per ASCII char, 1-4 bytes per Unicode char
- **utf-16**: 2 or 4 bytes per char (most common, used by rust-analyzer)
- **utf-32**: 4 bytes per char (rare)

When Telescope requests cursor position without specifying encoding, Neovim can't determine which client's encoding rules to use, hence the warning.

### Why Master Branch is Safe
- Telescope master branch is stable and well-tested
- Maintains backward compatibility with older Neovim versions
- Actively maintained with regular security and bug fixes
- Production-ready (used by many in daily workflows)

### Alternative Solutions Considered
1. **Downgrade Neovim to 0.10.x** - Rejected: loses newer features and fixes
2. **Pin to specific master commit** - Rejected: misses future fixes
3. **Fork and patch 0.1.x** - Rejected: maintenance burden

---

## Verification

### Test Procedure
1. Open Neovim with a Rust project
2. Navigate to any function call
3. Press `gd` to trigger go-to-definition
4. Observe: warning should not appear
5. Verify: definition jump still works correctly

### Expected Behavior
- No warning popup appears
- Go-to-definition works seamlessly
- All other LSP features remain functional
- Works with multiple LSP clients if present

---

## Impact Assessment

### Severity
**Low** - Cosmetic warning, functionality not affected

### Risk
**Minimal** - Telescope master is production-grade and backward compatible

### Benefits
- Eliminates user-facing warning
- Future-proofs configuration for Neovim 0.11+
- Gains access to newer Telescope features and fixes
- Aligns with Neovim's evolving LSP API standards

---

## Lessons Learned

### Plugin Version Management
1. **Stay current with major version updates** - Breaking changes in Neovim require plugin updates
2. **Monitor plugin changelogs** - Especially for core dependencies like Telescope
3. **Test after Neovim upgrades** - Verify all keymaps and LSP functionality

### Branch Strategy
- **0.1.x branch**: Stable but frozen, no new fixes
- **master branch**: Active development, receives compatibility updates
- For production use with latest Neovim: master is often safer than old stable

### LSP Position Encoding Best Practices
- Always specify position encoding when calling LSP utility functions
- Be aware of multiple LSP clients with different encodings
- Test thoroughly when mixing LSP servers (e.g., rust-analyzer + efm-langserver)

---

## References

### Related Files
- `init.lua:827` - Telescope plugin configuration
- `init.lua:1004` - `gd` keymap definition
- `init.lua:283-415` - Rustaceanvim configuration (correctly excludes rust-analyzer from Mason)

### Commits
- `905708e` - fix(telescope): update to master branch for nvim 0.11 compatibility
- `a17d611` - (Telescope upstream) fix(lsp): add default position encoding

### Documentation
- [Neovim 0.11 Release Notes](https://github.com/neovim/neovim/releases/tag/v0.11.0)
- [Telescope.nvim Issues](https://github.com/nvim-telescope/telescope.nvim/issues)
- [LSP Position Encoding Spec](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#position)

---

## Future Considerations

### Monitoring
- Watch for Telescope stable releases (potential future 0.2.x branch)
- Track Neovim LSP API evolution in release notes
- Consider pinning to specific commits in production if stability is critical

### Potential Improvements
1. Add health check for plugin version compatibility
2. Document expected Telescope version in project README
3. Consider adding automated testing for LSP functionality
4. Monitor for additional Neovim 0.11+ breaking changes

### Scale Deployment Notes
When replicating across multiple systems:
1. Ensure all systems are on Neovim 0.11+
2. Update Telescope configuration uniformly
3. Test with each language server in use (not just rust-analyzer)
4. Document any environment-specific variations
5. Consider using lock files (lazy-lock.json) for reproducible installations
