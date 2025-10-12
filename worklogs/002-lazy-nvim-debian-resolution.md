# Worklog 002: lazy.nvim Debian Bootstrap Resolution

**Date:** 2025-10-11
**Issue:** Persistent "module 'lazy' not found" on Debian after enhanced bootstrap fix
**Status:** ✅ Resolved
**Related:** Worklog 001 (`43a8d84`)

---

## Problem Statement

Despite implementing enhanced bootstrap error handling in Worklog 001, user "zazer" on Debian continued experiencing:
```
Error: module 'lazy' not found
```

The enhanced bootstrap code with robust error handling didn't resolve the underlying issue.

---

## Root Cause

**Corrupted/Incomplete Initial Installation**
- Previous failed bootstrap attempts left partial or corrupted files in `~/.local/share/nvim/lazy/lazy.nvim/`
- Neovim detected the directory exists → skipped bootstrap logic
- Directory lacked necessary Lua modules → "module not found" error
- Enhanced error handling never triggered because bootstrap was skipped

---

## Solution: Clean State + Fresh Install

### Step 1: Pull Updated Configuration
```bash
config pull
```
- Ensures latest `init.lua` with enhanced bootstrap (from Worklog 001)

### Step 2: Remove Corrupted Installation
```bash
rm -rf ~/.local/share/nvim/lazy/lazy.nvim
```
- Removes partial/corrupted lazy.nvim directory
- Forces bootstrap logic to run on next launch

### Step 3: Launch Neovim
```bash
nvim
```
- Bootstrap detects missing lazy.nvim
- Shows "Bootstrapping lazy.nvim plugin manager..." notification
- Clones fresh copy from GitHub
- Verifies installation succeeded
- Shows "lazy.nvim installed successfully!" notification

---

## Key Insights

### Why Enhanced Error Handling Wasn't Enough

1. **Directory Existence Check** (`init.lua:226`)
   ```lua
   if not vim.loop.fs_stat(lazypath) then
     -- bootstrap code
   end
   ```
   - If directory exists, bootstrap is skipped entirely
   - Corrupted installs pass this check but fail later

2. **Bootstrap vs Runtime Failure**
   - Enhanced error handling only applies during bootstrap
   - Corrupted installations fail at `require('lazy')` (line 249)
   - This happens AFTER bootstrap check → no enhanced errors shown

3. **State Persistence**
   - Failed attempts leave filesystem artifacts
   - Partial clones appear "installed" to existence checks
   - Network interruptions, permission errors can cause this

### Why rm + Reclone Works

- **Removes Ambiguous State:** Directory either doesn't exist (bootstrap) or is complete (runtime)
- **Forces Fresh Install:** Bootstrap logic guaranteed to run
- **Leverages Enhanced Error Handling:** If clone fails again, detailed errors now surface
- **Validates Installation:** Post-clone verification ensures completeness

---

## Diagnostic Steps (For Future Issues)

### Phase 1: Verify Prerequisites
```bash
# Check git availability
which git

# Check directory permissions
mkdir -p ~/.local/share/nvim/lazy && echo "OK" || echo "FAIL"

# Verify network access
curl -I https://github.com
```

### Phase 2: Clean State
```bash
# Remove potentially corrupted installation
rm -rf ~/.local/share/nvim/lazy/lazy.nvim

# Verify removal
ls ~/.local/share/nvim/lazy/
```

### Phase 3: Fresh Bootstrap
```bash
# Launch Neovim (triggers bootstrap)
nvim

# Expected: "Bootstrapping..." → "installed successfully!"
```

### Phase 4: Manual Bootstrap (Last Resort)
```bash
# If automatic bootstrap fails
git clone --filter=blob:none --branch=stable \
  https://github.com/folke/lazy.nvim.git \
  ~/.local/share/nvim/lazy/lazy.nvim
```

---

## Validation

### Success Indicators
- ✅ "Bootstrapping lazy.nvim plugin manager..." notification appears
- ✅ "lazy.nvim installed successfully!" confirmation
- ✅ Neovim launches without errors
- ✅ Plugins load correctly

### Verification Commands
```bash
# Check installation
ls ~/.local/share/nvim/lazy/lazy.nvim/lua/

# Verify Neovim can load lazy
nvim --headless -c "lua require('lazy')" -c "quit" 2>&1
```

---

## Lessons Learned

### For Bootstrap Logic

1. **Existence ≠ Validity**
   - Directory presence doesn't guarantee functional installation
   - Add integrity checks beyond `fs_stat()`
   - Consider checksums or marker files

2. **Fail-Safe Recovery**
   - Document clean-state procedure for users
   - Consider automatic corruption detection
   - Provide "nuclear option" for broken states

3. **State Machine Clarity**
   - Three states: Not Installed, Installing, Installed
   - Current code only handles: Not Installed vs Present
   - Missing: validity verification for "Present" state

### For Multi-System Deployments

1. **Clean First Launch Crucial**
   - First install sets baseline for all future operations
   - Corrupted first install causes cascading issues
   - Worth investing in robust initial bootstrap

2. **Network Resilience**
   - Transient network failures can corrupt installations
   - Consider retry logic or resumable clones
   - `--filter=blob:none` helps but doesn't solve all cases

3. **User Communication**
   - Enhanced error messages help (Worklog 001)
   - Recovery procedures equally important
   - Document "when all else fails" steps

### For Shared Dotfiles

1. **Per-User State Can Diverge**
   - One user's working config ≠ another's working state
   - Filesystem state not in version control
   - Document state reset procedures

2. **Bootstrap Must Be Idempotent**
   - Safe to run multiple times
   - Detects and handles partial completions
   - Recovers from failures gracefully

---

## Future Improvements

### Potential Enhancements

1. **Installation Integrity Check**
   ```lua
   -- After existence check
   if vim.loop.fs_stat(lazypath) then
     -- Verify lua/lazy.nvim/init.lua exists
     local init_file = lazypath .. '/lua/lazy/init.lua'
     if vim.loop.fs_stat(init_file) == nil then
       -- Corrupted install, remove and rebootstrap
       vim.fn.delete(lazypath, 'rf')
       -- Fall through to bootstrap
     end
   end
   ```

2. **Automated Recovery**
   - Detect `require('lazy')` failure
   - Auto-trigger clean + rebootstrap
   - Limit retry attempts to prevent loops

3. **Health Check Command**
   - `:checkhealth lazy` equivalent
   - Validates installation integrity
   - Suggests fixes for common issues

---

## Related Files

- `init.lua:225-263` - Enhanced bootstrap code (Worklog 001)
- `init.lua:226` - Directory existence check (skip bootstrap if exists)
- `init.lua:249` - `require('lazy')` call (fails if corrupted)

---

## Commands Reference

### For Users Experiencing This Issue

```bash
# 1. Update config
config pull

# 2. Remove corrupted installation
rm -rf ~/.local/share/nvim/lazy/lazy.nvim

# 3. Launch Neovim (auto-bootstraps)
nvim
```

### For Debugging

```bash
# Check lazy.nvim structure
find ~/.local/share/nvim/lazy/lazy.nvim -type f | head -20

# Verify Lua modules present
ls ~/.local/share/nvim/lazy/lazy.nvim/lua/lazy/

# Test require from command line
nvim --headless -c "lua print(vim.inspect(require('lazy')))" -c "quit"
```

---

## Summary

**Problem:** Enhanced bootstrap didn't fix Debian issue
**Root Cause:** Corrupted installation bypassed bootstrap logic
**Solution:** `rm -rf` + fresh clone
**Key Learning:** Directory existence ≠ functional installation

**Success Criteria Met:**
- ✅ Neovim launches without errors on Debian
- ✅ Bootstrap process completes successfully
- ✅ Enhanced error handling available for future failures
- ✅ Clear recovery procedure documented

---

## References

- Worklog 001: Enhanced bootstrap error handling
- [lazy.nvim installation docs](https://github.com/folke/lazy.nvim#-installation)
- `init.lua:225-263` - Bootstrap implementation
