# Dawn Theme Update Strategy for Nitro Theme

## Overview

This document outlines the strategy for keeping the Nitro theme synchronized with upstream changes from Shopify's Dawn theme while maintaining custom modifications.

**Current Setup:**
- **Child Theme:** Nitro (based on Dawn 12)
- **Parent Theme:** Shopify Dawn
- **Repository:** https://github.com/mkutev/nitro-theme.git

## Key Custom Modifications in Nitro

Based on the theme analysis, Nitro includes these custom features:
- Custom badge system (3 configurable badges with text/image options)
- Custom badge positioning and layout controls
- Dynamic image badge functionality
- Modified color schemes and branding

---

## Update Strategy

### Initial Setup (One-Time Configuration)

#### 1. Add Dawn as an Upstream Remote

```bash
# Add the official Dawn repository as a remote
git remote add dawn https://github.com/Shopify/dawn.git

# Fetch all Dawn branches and tags
git fetch dawn

# Verify the remote was added
git remote -v
```

#### 2. Create a Tracking Branch for Dawn

```bash
# Create a local branch to track Dawn's main branch
git checkout -b dawn-upstream dawn/main

# Return to your main branch
git checkout main
```

#### 3. Tag Your Current State

```bash
# Tag the current state before any updates
git tag -a pre-dawn-sync-v1 -m "State before first Dawn sync"
git push origin --tags
```

---

## Update Workflow

### Phase 1: Preparation

**Before starting any update:**

1. **Create a backup branch:**
   ```bash
   git checkout -b backup/pre-update-$(date +%Y%m%d)
   git push origin backup/pre-update-$(date +%Y%m%d)
   git checkout main
   ```

2. **Document current customizations:**
   - Review custom badge settings in `config/settings_schema.json` (lines 1272-1622)
   - Document any custom sections, snippets, or liquid templates
   - List modified asset files (CSS, JavaScript)

3. **Create a feature branch for the update:**
   ```bash
   git checkout -b feature/dawn-update-$(date +%Y%m%d)
   ```

### Phase 2: Fetch and Review Dawn Changes

4. **Update Dawn remote:**
   ```bash
   # Fetch latest Dawn changes
   git fetch dawn

   # Update your Dawn tracking branch
   git checkout dawn-upstream
   git merge dawn/main
   git checkout feature/dawn-update-$(date +%Y%m%d)
   ```

5. **Review what changed in Dawn:**
   ```bash
   # See commits since your last sync (replace LAST_SYNC_TAG with your tag)
   git log --oneline --graph LAST_SYNC_TAG..dawn/main

   # Get detailed view of changes
   git log --stat LAST_SYNC_TAG..dawn/main

   # Review specific file changes
   git diff LAST_SYNC_TAG..dawn/main -- path/to/file
   ```

6. **Identify affected areas:**
   ```bash
   # List all changed files
   git diff --name-only LAST_SYNC_TAG..dawn/main

   # Filter for critical files
   git diff --name-only LAST_SYNC_TAG..dawn/main | grep -E '(settings_schema|config|sections)'
   ```

### Phase 3: Merge Strategy

**Choose the appropriate merge approach:**

#### Option A: Cherry-Pick Specific Changes (Recommended for Minor Updates)

Best for: Bug fixes, security patches, or specific feature improvements

```bash
# Review commits you want to apply
git log --oneline LAST_SYNC_TAG..dawn/main

# Cherry-pick specific commits
git cherry-pick <commit-hash>

# If conflicts occur, resolve them (see Phase 4)
```

#### Option B: Three-Way Merge (For Major Version Updates)

Best for: Major Dawn version updates

```bash
# Merge Dawn changes into your feature branch
git merge dawn/main -m "Merge Dawn updates from $(date +%Y-%m-%d)"

# Resolve conflicts (see Phase 4)
```

#### Option C: Manual File-by-File Review (Most Conservative)

Best for: When you want maximum control

```bash
# For each changed file:
git show dawn/main:path/to/file > /tmp/dawn-version.liquid
# Manually compare and apply changes to your version
```

### Phase 4: Conflict Resolution

When conflicts occur:

1. **Identify conflict files:**
   ```bash
   git status
   ```

2. **For each conflict, categorize it:**

   **Type A: Pure Dawn updates (no custom changes)**
   - Action: Accept Dawn's version
   ```bash
   git checkout --theirs path/to/file
   git add path/to/file
   ```

   **Type B: Pure Nitro customizations (no Dawn changes needed)**
   - Action: Keep your version
   ```bash
   git checkout --ours path/to/file
   git add path/to/file
   ```

   **Type C: Both Dawn and Nitro modified the same area**
   - Action: Manual merge required
   ```bash
   # Open in your editor and resolve conflicts manually
   # Look for <<<<<<< HEAD, =======, >>>>>>> markers
   code path/to/file

   # After resolving, stage the file
   git add path/to/file
   ```

3. **Critical files requiring careful review:**
   - `config/settings_schema.json` - Preserve custom badge settings (lines 1272-1622)
   - `snippets/` - Check for custom badge rendering logic
   - `sections/` - Preserve any custom section modifications
   - `assets/` - Merge CSS/JS changes carefully to preserve custom styling

### Phase 5: Testing

**Pre-deployment testing checklist:**

1. **Theme validation:**
   ```bash
   # If you have Shopify CLI installed
   shopify theme check
   ```

2. **Visual regression testing:**
   - [ ] Homepage renders correctly
   - [ ] Product pages display properly
   - [ ] Collection pages work as expected
   - [ ] Custom badges appear correctly
   - [ ] Cart functionality works
   - [ ] Checkout flow is functional
   - [ ] Mobile responsiveness maintained
   - [ ] Custom color schemes intact

3. **Feature testing:**
   - [ ] All custom badge configurations work
   - [ ] Dynamic image badges function
   - [ ] Badge positioning/layout controls work
   - [ ] Navigation and search work
   - [ ] Product variants function correctly
   - [ ] Forms submit properly

4. **Deploy to development theme:**
   ```bash
   # Using Shopify CLI
   shopify theme push --development
   ```

5. **Cross-browser testing:**
   - [ ] Chrome/Edge
   - [ ] Firefox
   - [ ] Safari
   - [ ] Mobile browsers

### Phase 6: Deployment

If testing passes:

1. **Commit the merge:**
   ```bash
   git commit -m "Merge Dawn updates - $(date +%Y-%m-%d)

   - Updated from Dawn version X.X.X
   - Resolved conflicts in: [list files]
   - Preserved custom features: [list features]
   - Tested on development theme
   "
   ```

2. **Tag the new version:**
   ```bash
   git tag -a v1.x.x -m "Nitro theme with Dawn updates from $(date +%Y-%m-%d)"
   ```

3. **Merge to main:**
   ```bash
   git checkout main
   git merge feature/dawn-update-$(date +%Y%m%d)
   git push origin main --tags
   ```

4. **Deploy to production:**
   ```bash
   shopify theme push --live
   ```

### Phase 7: Documentation

After successful deployment:

1. **Update changelog:**
   Create/update `CHANGELOG.md` with:
   - Dawn version merged
   - Date of merge
   - List of new features/fixes from Dawn
   - Any breaking changes
   - Custom features preserved

2. **Tag last sync point:**
   ```bash
   git tag -a last-dawn-sync -f -m "Last Dawn sync on $(date +%Y-%m-%d)"
   git push origin last-dawn-sync -f
   ```

---

## Conflict Resolution Guide

### Common Conflict Scenarios

#### Scenario 1: Settings Schema Conflicts

**File:** `config/settings_schema.json`

**Strategy:**
1. Dawn may add new settings sections - these can usually be added without conflict
2. Your custom badge settings (lines 1272-1622) should be preserved
3. If Dawn modifies existing sections, manually merge the changes

**Example resolution:**
```json
{
  "name": "t:settings_schema.badges.name",
  "settings": [
    // Dawn's original badge settings
    ...
  ]
},
{
  "name": "Custom Badges",  // Your custom section - preserve this
  "settings": [
    // Your custom badge configurations
    ...
  ]
}
```

#### Scenario 2: Section File Updates

**Files:** `sections/*.liquid`

**Strategy:**
1. If you haven't modified the section: accept Dawn's version
2. If you added custom features: manually merge
3. Use `git diff` to see both versions side-by-side

**Process:**
```bash
# See your changes vs Dawn's
git diff HEAD dawn/main -- sections/product-grid.liquid

# Use a merge tool for visual comparison
git mergetool
```

#### Scenario 3: Asset Files (CSS/JS)

**Files:** `assets/*.css`, `assets/*.js`

**Strategy:**
1. Custom styling for badges should be isolated if possible
2. Dawn's base styles should be accepted
3. Re-apply your custom styles on top

**Best practice:**
- Keep custom CSS in separate files or clearly marked sections
- Use CSS custom properties for theme modifications
- Comment custom code blocks clearly

#### Scenario 4: Snippet Modifications

**Files:** `snippets/*.liquid`

**Strategy:**
1. If snippet doesn't exist in Dawn: keep your version
2. If both modified: carefully merge logic
3. Test thoroughly as snippets are reused across sections

---

## Monitoring Dawn Updates

### Set Up Update Notifications

1. **GitHub Watch:**
   - Go to https://github.com/Shopify/dawn
   - Click "Watch" → "Custom" → Select "Releases"

2. **Check for updates monthly:**
   ```bash
   git fetch dawn
   git log --oneline main..dawn/main
   ```

3. **Review Dawn's changelog:**
   - https://github.com/Shopify/dawn/releases

### Recommended Update Frequency

- **Security fixes:** Immediately
- **Bug fixes:** Within 1-2 weeks
- **Minor features:** Monthly review
- **Major versions:** Quarterly review, plan 2-3 weeks for testing

---

## Rollback Plan

If an update causes issues in production:

### Immediate Rollback

```bash
# Revert to previous commit
git revert HEAD

# Or reset to previous tag
git reset --hard <previous-tag>
git push origin main --force

# Redeploy previous version
shopify theme push --live
```

### Investigate and Fix

```bash
# Create a hotfix branch from production
git checkout -b hotfix/dawn-update-issues main

# Fix the issues
# ... make changes ...

# Deploy hotfix
git commit -m "Hotfix: Resolve Dawn update issues"
git checkout main
git merge hotfix/dawn-update-issues
git push origin main
```

---

## Best Practices

### Code Organization

1. **Clearly mark custom code:**
   ```liquid
   {%- comment -%} CUSTOM: Nitro badge feature - START {%- endcomment -%}
   <!-- Your custom code -->
   {%- comment -%} CUSTOM: Nitro badge feature - END {%- endcomment -%}
   ```

2. **Use separate files for major customizations:**
   - `snippets/nitro-custom-badges.liquid`
   - `assets/nitro-custom.css`
   - `assets/nitro-custom.js`

3. **Document modifications:**
   - Maintain a `CUSTOMIZATIONS.md` file listing all changes from Dawn
   - Update it whenever you add custom features

### Version Control

1. **Never work directly on main:**
   - Always use feature branches
   - Require PR reviews for Dawn updates

2. **Tag all significant states:**
   - Before updates
   - After successful updates
   - Production releases

3. **Meaningful commit messages:**
   ```
   feat: Add custom badge system for product cards
   fix: Resolve conflict in settings_schema.json
   merge: Dawn v12.1.0 updates
   ```

### Testing

1. **Maintain a checklist:** Keep a testing checklist specific to your customizations
2. **Use development themes:** Always test on non-production themes first
3. **Document test results:** Keep notes on what was tested and results

---

## Tools and Resources

### Useful Git Commands

```bash
# Compare your current state with Dawn
git diff main dawn/main

# See files that differ
git diff --name-only main dawn/main

# Interactive merge tool
git mergetool --tool=vimdiff

# Show file from specific branch/commit
git show dawn/main:path/to/file

# Find when a line was changed
git blame path/to/file

# Search commit history
git log --all --grep="badge"
```

### Shopify CLI Commands

```bash
# Check theme for errors
shopify theme check

# Push to development theme
shopify theme push --development

# Pull current live theme
shopify theme pull --live

# Compare local with live
shopify theme compare
```

### Recommended Tools

- **VS Code Extensions:**
  - Shopify Liquid
  - GitLens
  - Diff View

- **Merge Tools:**
  - Beyond Compare
  - Kaleidoscope
  - Meld
  - VS Code built-in diff

---

## Quick Reference Checklist

### Before Update
- [ ] Create backup branch
- [ ] Document current customizations
- [ ] Fetch latest Dawn changes
- [ ] Review Dawn changelog
- [ ] Create feature branch

### During Update
- [ ] Choose merge strategy
- [ ] Merge/cherry-pick changes
- [ ] Resolve conflicts carefully
- [ ] Preserve custom features
- [ ] Test locally

### After Update
- [ ] Run theme validation
- [ ] Deploy to development theme
- [ ] Complete testing checklist
- [ ] Get stakeholder approval
- [ ] Deploy to production
- [ ] Update documentation
- [ ] Tag release

---

## Contact and Support

**For Dawn-specific issues:**
- GitHub Issues: https://github.com/Shopify/dawn/issues
- Shopify Community: https://community.shopify.com/

**For Theme Development:**
- Shopify Theme Docs: https://shopify.dev/themes
- Liquid Reference: https://shopify.dev/api/liquid

---

**Last Updated:** 2025-12-01
**Nitro Theme Version:** Based on Dawn 12
**Maintained by:** Momchil Kutev
