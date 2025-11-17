---
name: github-pr-creator
version: 1.0.0
description: Create GitHub Pull Requests from feature branches with issue references and template adherence
author: Chris Phillipson
tags: [github, automation, pull-request, workflow, issue-management]
---

# GitHub PR Creator Skill

Automates the creation of GitHub Pull Requests from feature branches with intelligent template population, issue references, and comprehensive change analysis.

## When to Use This Skill

Use this skill when:

- You're ready to create a PR after implementing changes on a feature branch
- You want to ensure PR follows project template standards
- You need automatic extraction of changes, commits, and issue context
- You want intelligent auto-fill of deterministic PR template fields
- You're completing work started with github-issue-implementer skill

## Capabilities

This skill will:

1. Verify you're on a feature branch (not main/master)
2. Extract issue number from branch name or user input
3. Fetch issue details and context from GitHub
4. Analyze git changes and commit history
5. Auto-fill PR template with contextual information
6. Prompt for manual input on subjective fields
7. Create PR with complete, well-formatted content
8. Display PR URL and summary

## Branch Prerequisites

This skill assumes you are already on a feature branch. It works seamlessly after:

- Using github-issue-implementer to create a branch and implement changes
- Manually creating a branch and making changes
- Switching to an existing feature branch with uncommitted or committed work

**The skill will NOT create branches** - use github-issue-implementer for that.

## Instructions

When this skill is invoked, follow this workflow:

### Step 1: Verify Current Branch

Check that the user is on a feature branch:

```bash
# Get current branch name
current_branch=$(git branch --show-current)

# Check if on main/master
if [[ "$current_branch" == "main" || "$current_branch" == "master" ]]; then
    echo "❌ Cannot create PR from main/master branch"
    echo "Please switch to a feature branch first"
    echo "Tip: Use github-issue-implementer to create a branch from an issue"
    exit 1
fi

echo "✓ On branch: $current_branch"
```

### Step 2: Extract or Request Issue Number

Try to extract issue number from branch name, or ask the user:

```bash
# Try to extract issue number from branch name patterns:
# - feature/123-description
# - fix/issue-456-description
# - feature/description-references-789

issue_number=""

# Pattern 1: prefix/123-description
if [[ "$current_branch" =~ ^[a-z]+/([0-9]+)- ]]; then
    issue_number="${BASH_REMATCH[1]}"
# Pattern 2: prefix/issue-123-description
elif [[ "$current_branch" =~ issue-([0-9]+) ]]; then
    issue_number="${BASH_REMATCH[1]}"
fi

# If not found in branch name, ask user
if [[ -z "$issue_number" ]]; then
    # Use AskUserQuestion tool
    echo "Could not extract issue number from branch name: $current_branch"
    # Ask: "What GitHub issue does this PR address?"
    # Provide option to enter issue number or skip if no related issue
fi

echo "✓ Issue number: #$issue_number"
```

### Step 3: Fetch Issue Details

If an issue number was provided, fetch its details:

```bash
# Fetch issue information
if [[ -n "$issue_number" ]]; then
    issue_title=$(gh issue view "$issue_number" --json title -q .title 2>/dev/null)
    issue_body=$(gh issue view "$issue_number" --json body -q .body 2>/dev/null)
    issue_labels=$(gh issue view "$issue_number" --json labels -q '.labels[].name' 2>/dev/null)
    issue_url=$(gh issue view "$issue_number" --json url -q .url 2>/dev/null)

    if [[ -z "$issue_title" ]]; then
        echo "⚠ Could not fetch issue #$issue_number (may not exist or no access)"
        echo "Proceeding without issue context"
    else
        echo "✓ Fetched issue: $issue_title"
    fi
fi
```

### Step 4: Analyze Git Changes

Collect information about changes made:

```bash
# Get base branch (usually main)
base_branch="main"
if git show-ref --verify --quiet refs/heads/master; then
    base_branch="master"
fi

# Get commit count
commit_count=$(git rev-list --count "$base_branch..HEAD")
echo "✓ Commits: $commit_count"

# Get commit messages
commits=$(git log "$base_branch..HEAD" --pretty=format:"- %s" --reverse)

# Get file changes summary
files_changed=$(git diff "$base_branch...HEAD" --name-only | wc -l | tr -d ' ')
insertions=$(git diff "$base_branch...HEAD" --shortstat | sed -n 's/.* \([0-9]\+\) insertion.*/\1/p')
deletions=$(git diff "$base_branch...HEAD" --shortstat | sed -n 's/.* \([0-9]\+\) deletion.*/\1/p')

echo "✓ Files changed: $files_changed"
echo "✓ Insertions: ${insertions:-0}, Deletions: ${deletions:-0}"

# Get detailed file changes with context
file_changes=$(git diff "$base_branch...HEAD" --name-status)
```

### Step 5: Determine PR Type

Analyze changes to determine PR type:

```bash
# Determine type from:
# 1. Issue labels (if available)
# 2. Branch prefix
# 3. Commit messages

pr_type=""

# Check issue labels first
if [[ "$issue_labels" =~ bug ]]; then
    pr_type="bug"
elif [[ "$issue_labels" =~ enhancement|feature ]]; then
    pr_type="feature"
elif [[ "$issue_labels" =~ documentation ]]; then
    pr_type="docs"
elif [[ "$issue_labels" =~ security ]]; then
    pr_type="security"
fi

# Check branch prefix if type not determined
if [[ -z "$pr_type" ]]; then
    if [[ "$current_branch" =~ ^fix/ ]]; then
        pr_type="bug"
    elif [[ "$current_branch" =~ ^feature/ ]]; then
        pr_type="feature"
    elif [[ "$current_branch" =~ ^docs/ ]]; then
        pr_type="docs"
    elif [[ "$current_branch" =~ ^refactor/ ]]; then
        pr_type="refactor"
    elif [[ "$current_branch" =~ ^test/ ]]; then
        pr_type="test"
    fi
fi

# Check commit messages as fallback
if [[ -z "$pr_type" ]]; then
    if echo "$commits" | grep -qi "fix:"; then
        pr_type="bug"
    elif echo "$commits" | grep -qi "feat:"; then
        pr_type="feature"
    elif echo "$commits" | grep -qi "docs:"; then
        pr_type="docs"
    fi
fi

# Default to feature if still unknown
pr_type="${pr_type:-feature}"

echo "✓ Detected PR type: $pr_type"
```

### Step 6: Auto-Fill PR Template (Hybrid Approach)

Read the PR template and auto-fill deterministic fields:

```bash
# Read the template
template_path="skills/github-pr-creator/templates/pull_request_template.md"
template_content=$(cat "$template_path")

# Start building PR body
pr_body=""

# === SECTION 1: Description (Auto-fill) ===
pr_body+="# Pull Request\n\n"
pr_body+="## Description\n\n"

# Generate description from commits and issue
if [[ -n "$issue_title" ]]; then
    pr_body+="This PR implements: $issue_title\n\n"
fi

# Add brief summary based on commits
pr_body+="### Summary of Changes\n\n"
pr_body+="$commits\n\n"

# Add issue reference
if [[ -n "$issue_number" ]]; then
    pr_body+="Fixes #$issue_number\n\n"
else
    pr_body+="Fixes #(issue number)\n\n"
fi

# === SECTION 2: Type of Change (Auto-select based on detection) ===
pr_body+="## Type of Change\n\n"
pr_body+="Please delete options that are not relevant.\n\n"

# Auto-check the detected type
case "$pr_type" in
    bug)
        pr_body+="- [x] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [ ] ✨ New feature (non-breaking change which adds functionality)\n"
        ;;
    feature)
        pr_body+="- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [x] ✨ New feature (non-breaking change which adds functionality)\n"
        ;;
    docs)
        pr_body+="- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [ ] ✨ New feature (non-breaking change which adds functionality)\n"
        pr_body+="- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)\n"
        pr_body+="- [x] 📚 Documentation update\n"
        ;;
    security)
        pr_body+="- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [ ] ✨ New feature (non-breaking change which adds functionality)\n"
        pr_body+="- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)\n"
        pr_body+="- [ ] 📚 Documentation update\n"
        pr_body+="- [ ] 🔧 Configuration change\n"
        pr_body+="- [ ] 🎨 Style/formatting change\n"
        pr_body+="- [ ] ♻️ Code refactoring\n"
        pr_body+="- [ ] ⚡ Performance improvement\n"
        pr_body+="- [x] 🔒 Security improvement\n"
        ;;
    refactor)
        pr_body+="- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [ ] ✨ New feature (non-breaking change which adds functionality)\n"
        pr_body+="- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)\n"
        pr_body+="- [ ] 📚 Documentation update\n"
        pr_body+="- [ ] 🔧 Configuration change\n"
        pr_body+="- [ ] 🎨 Style/formatting change\n"
        pr_body+="- [x] ♻️ Code refactoring\n"
        ;;
    *)
        # Default - no auto-check
        pr_body+="- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)\n"
        pr_body+="- [ ] ✨ New feature (non-breaking change which adds functionality)\n"
        ;;
esac

pr_body+="- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)\n"
pr_body+="- [ ] 📚 Documentation update\n"
pr_body+="- [ ] 🔧 Configuration change\n"
pr_body+="- [ ] 🎨 Style/formatting change\n"
pr_body+="- [ ] ♻️ Code refactoring\n"
pr_body+="- [ ] ⚡ Performance improvement\n"
pr_body+="- [ ] 🔒 Security improvement\n\n"

# === SECTION 3: Changes Made (Auto-fill from file changes) ===
pr_body+="## Changes Made\n\n"

# Group changes by type (Added, Modified, Deleted)
added_files=$(echo "$file_changes" | grep "^A" | cut -f2- || echo "")
modified_files=$(echo "$file_changes" | grep "^M" | cut -f2- || echo "")
deleted_files=$(echo "$file_changes" | grep "^D" | cut -f2- || echo "")

if [[ -n "$added_files" ]]; then
    pr_body+="### Added\n"
    while IFS= read -r file; do
        [[ -n "$file" ]] && pr_body+="- [ ] $file\n"
    done <<< "$added_files"
    pr_body+="\n"
fi

if [[ -n "$modified_files" ]]; then
    pr_body+="### Modified\n"
    while IFS= read -r file; do
        [[ -n "$file" ]] && pr_body+="- [ ] $file\n"
    done <<< "$modified_files"
    pr_body+="\n"
fi

if [[ -n "$deleted_files" ]]; then
    pr_body+="### Deleted\n"
    while IFS= read -r file; do
        [[ -n "$file" ]] && pr_body+="- [ ] $file\n"
    done <<< "$deleted_files"
    pr_body+="\n"
fi

# === SECTION 4: Testing (Prompt user) ===
pr_body+="## Testing\n\n"
pr_body+="Describe the tests that you ran to verify your changes.\n\n"
pr_body+="- [ ] Manual testing performed\n"
pr_body+="- [ ] Unit tests added/updated\n"
pr_body+="- [ ] Integration tests pass\n"
pr_body+="- [ ] Documentation tested\n\n"

# Rest of template with placeholders
pr_body+="### Test Environment\n\n"
pr_body+="- OS: [e.g., macOS 14.0, Ubuntu 22.04]\n"
pr_body+="- Fly.io region: [e.g., iad, lax, fra]\n"
pr_body+="- VM configuration: [e.g., shared-cpu-2x, 8GB]\n\n"

pr_body+="## Cost Impact\n\n"
pr_body+="- [x] No impact on Fly.io costs\n"
pr_body+="- [ ] Reduces costs (explain how)\n"
pr_body+="- [ ] Increases costs (explain why and how much)\n"
pr_body+="- [ ] Cost impact unknown/needs investigation\n\n"

pr_body+="## Security Considerations\n\n"
pr_body+="- [x] No security implications\n"
pr_body+="- [ ] Security improvements included\n"
pr_body+="- [ ] Potential security concerns (describe)\n"
pr_body+="- [ ] Security review requested\n\n"

pr_body+="## Breaking Changes\n\n"
pr_body+="If this introduces breaking changes, please describe:\n\n"
pr_body+="- What breaks\n"
pr_body+="- Migration path for users\n"
pr_body+="- Timeline for deprecation (if applicable)\n\n"

pr_body+="## Documentation\n\n"
pr_body+="- [x] Code is self-documenting\n"
pr_body+="- [ ] Comments added for complex logic\n"
pr_body+="- [ ] README updated\n"
pr_body+="- [ ] Documentation in \`/docs\` updated\n"
pr_body+="- [ ] Examples updated\n"
pr_body+="- [ ] CHANGELOG.md entry added\n\n"

pr_body+="## Checklist\n\n"
pr_body+="- [ ] My code follows the project's style guidelines\n"
pr_body+="- [ ] I have performed a self-review of my code\n"
pr_body+="- [ ] I have commented my code, particularly in hard-to-understand areas\n"
pr_body+="- [ ] I have made corresponding changes to the documentation\n"
pr_body+="- [ ] My changes generate no new warnings\n"
pr_body+="- [ ] I have added tests that prove my fix is effective or that my feature works\n"
pr_body+="- [ ] New and existing unit tests pass locally with my changes\n"
pr_body+="- [ ] Any dependent changes have been merged and published\n\n"

pr_body+="## Additional Context\n\n"
pr_body+="Add any other context about the pull request here.\n\n"

pr_body+="## Deployment Notes\n\n"
pr_body+="Special instructions for deploying these changes:\n\n"
pr_body+="- [x] Standard deployment\n"
pr_body+="- [ ] Requires manual intervention\n"
pr_body+="- [ ] Database migration needed\n"
pr_body+="- [ ] Configuration changes required\n"
pr_body+="- [ ] Secrets need to be updated\n\n"

pr_body+="## Reviewer Notes\n\n"
pr_body+="Anything specific you want reviewers to focus on:\n\n"
pr_body+="- Performance implications\n"
pr_body+="- Security considerations\n"
pr_body+="- Architecture decisions\n"
pr_body+="- Alternative approaches considered\n"
```

### Step 7: Prompt for Manual Fields

Use AskUserQuestion to gather information for subjective fields:

**Questions to ask:**

1. **Testing Details**
   - "What testing have you performed?"
   - Options: "Manual testing only", "Added unit tests", "Added integration tests", "Full test coverage"

2. **Security Impact**
   - "Are there any security considerations?"
   - Options: "No security implications", "Security improvements included", "Needs security review"

3. **Cost Impact** (if relevant to project)
   - "Does this change impact costs?"
   - Options: "No impact", "Reduces costs", "Increases costs", "Unknown"

4. **Breaking Changes**
   - "Does this introduce breaking changes?"
   - Options: "No breaking changes", "Yes, with migration path", "Yes, requires attention"

**Integrate answers into PR body** by updating the relevant sections with specific details provided.

### Step 8: Generate PR Title

Create a descriptive PR title:

```bash
# Use conventional commit format
pr_title=""

case "$pr_type" in
    bug)
        pr_title="fix: "
        ;;
    feature)
        pr_title="feat: "
        ;;
    docs)
        pr_title="docs: "
        ;;
    refactor)
        pr_title="refactor: "
        ;;
    test)
        pr_title="test: "
        ;;
    security)
        pr_title="security: "
        ;;
    *)
        pr_title="chore: "
        ;;
esac

# Add title from issue or generate from branch
if [[ -n "$issue_title" ]]; then
    # Clean up issue title
    clean_title=$(echo "$issue_title" | sed 's/\[.*\]:*//g' | sed 's/^ *//')
    pr_title+="$clean_title"
else
    # Generate from branch name
    branch_title=$(echo "$current_branch" | sed 's/^[^\/]*\///' | sed 's/-/ /g')
    pr_title+="$branch_title"
fi

# Limit to 72 characters (GitHub best practice)
pr_title=$(echo "$pr_title" | cut -c1-72)

echo "✓ PR title: $pr_title"
```

### Step 9: Create Pull Request

Use `gh` CLI to create the PR:

```bash
# Save PR body to temp file
temp_pr_file=$(mktemp)
echo -e "$pr_body" > "$temp_pr_file"

# Create PR
pr_url=$(gh pr create \
    --title "$pr_title" \
    --body-file "$temp_pr_file" \
    --base "$base_branch" \
    --head "$current_branch")

# Clean up temp file
rm "$temp_pr_file"

if [[ -n "$pr_url" ]]; then
    echo "✓ PR created: $pr_url"
else
    echo "❌ Failed to create PR"
    exit 1
fi
```

### Step 10: Provide Summary

Report back to the user:

```markdown
## PR Created Successfully

**URL:** <pr-url>
**Title:** <pr-title>
**Base:** <base-branch>
**Head:** <current-branch>

### Summary

- Issue: #<issue-number> (if applicable)
- Type: <pr-type>
- Files changed: <count>
- Commits: <count>

### Auto-Filled Sections

- ✓ Description with issue reference
- ✓ Type of change (detected as <type>)
- ✓ Changes made (from git diff)
- ✓ Commit history included

### Manual Review Needed

Please review and update:
- Testing details
- Test environment specifics
- Security considerations
- Cost impact assessment
- Breaking changes documentation
- Documentation checklist
- Additional context

You can edit the PR at: <pr-url>
```

## Example Usage

### Example 1: Feature Branch with Issue Reference

**User Request:**
> Use github-pr-creator skill

**Context:**
- On branch: `feature/transparent-secrets-management-with-sops`
- Issue: #24 "Transparent Secrets Management with SOPS + age"
- 6 commits made
- 8 files changed

**Skill Actions:**

1. Verify branch: `feature/transparent-secrets-management-with-sops` ✓
2. Extract issue number: #24
3. Fetch issue details: "Transparent Secrets Management with SOPS + age"
4. Analyze changes: 6 commits, 8 files (5 added, 3 modified)
5. Detect type: feature (from branch prefix and issue labels)
6. Auto-fill template:
   - Description: "This PR implements: Transparent Secrets Management with SOPS + age"
   - Issue reference: "Fixes #24"
   - Type: ✅ New feature
   - Changes: Lists all 8 files with status
7. Prompt for testing, security, cost details
8. Generate title: "feat: Transparent Secrets Management with SOPS + age"
9. Create PR
10. Return PR URL and summary

**Output:**

```text
✓ On branch: feature/transparent-secrets-management-with-sops
✓ Issue number: #24
✓ Fetched issue: Transparent Secrets Management with SOPS + age
✓ Commits: 6
✓ Files changed: 8
✓ Detected PR type: feature
✓ PR title: feat: Transparent Secrets Management with SOPS + age
✓ PR created: https://github.com/user/repo/pull/42

## PR Created Successfully

**URL:** https://github.com/user/repo/pull/42
**Title:** feat: Transparent Secrets Management with SOPS + age
**Base:** main
**Head:** feature/transparent-secrets-management-with-sops

### Summary

- Issue: #24
- Type: feature
- Files changed: 8
- Commits: 6

### Auto-Filled Sections

- ✓ Description with issue reference
- ✓ Type of change (detected as feature)
- ✓ Changes made (5 added, 3 modified)
- ✓ Commit history included

Please review the PR and update manual sections as needed.
```

### Example 2: Bug Fix Without Issue

**User Request:**
> Create a PR for my bug fix

**Context:**
- On branch: `fix/ruby-extension-timeout`
- No issue number in branch name
- 3 commits made

**Skill Actions:**

1. Verify branch: `fix/ruby-extension-timeout` ✓
2. Cannot extract issue number from branch
3. Ask user: "What GitHub issue does this PR address?"
4. User provides: #25
5. Fetch issue details
6. Analyze changes: 3 commits, 2 files modified
7. Detect type: bug (from branch prefix)
8. Auto-fill template
9. Generate title: "fix: Ruby extension timeout handling"
10. Create PR

**Output:**

```text
✓ PR created: https://github.com/user/repo/pull/43

Issue: #25
Type: bug fix
Files changed: 2
Commits: 3
```

### Example 3: Documentation Update

**User Request:**
> Ready to submit my docs changes

**Context:**
- On branch: `docs/mise-integration-guide`
- 2 commits
- 1 markdown file added

**Skill Actions:**

1. Verify branch ✓
2. Extract issue number: none
3. Ask user for issue number: user skips
4. Analyze changes: 2 commits, 1 file added
5. Detect type: docs
6. Auto-fill template without issue reference
7. Generate title: "docs: mise integration guide"
8. Create PR

## Helper Functions

When implementing this skill, use these patterns:

### Extract Issue Number from Branch

```bash
extract_issue_from_branch() {
    local branch_name="$1"
    local issue_num=""

    # Pattern 1: prefix/123-description
    if [[ "$branch_name" =~ ^[a-z]+/([0-9]+)- ]]; then
        issue_num="${BASH_REMATCH[1]}"
    # Pattern 2: prefix/issue-123-description
    elif [[ "$branch_name" =~ issue-([0-9]+) ]]; then
        issue_num="${BASH_REMATCH[1]}"
    # Pattern 3: prefix/description-123
    elif [[ "$branch_name" =~ -([0-9]+)$ ]]; then
        issue_num="${BASH_REMATCH[1]}"
    fi

    echo "$issue_num"
}
```

### Detect PR Type

```bash
detect_pr_type() {
    local branch="$1"
    local labels="$2"
    local commits="$3"

    # Priority 1: Issue labels
    if [[ "$labels" =~ bug ]]; then
        echo "bug"
        return
    elif [[ "$labels" =~ enhancement|feature ]]; then
        echo "feature"
        return
    fi

    # Priority 2: Branch prefix
    case "$branch" in
        fix/*) echo "bug" ;;
        feature/*) echo "feature" ;;
        docs/*) echo "docs" ;;
        refactor/*) echo "refactor" ;;
        test/*) echo "test" ;;
        security/*) echo "security" ;;
        *) echo "feature" ;;  # Default
    esac
}
```

### Generate Conventional Commit Title

```bash
generate_pr_title() {
    local pr_type="$1"
    local issue_title="$2"
    local branch_name="$3"

    local prefix=""
    case "$pr_type" in
        bug) prefix="fix: " ;;
        feature) prefix="feat: " ;;
        docs) prefix="docs: " ;;
        refactor) prefix="refactor: " ;;
        test) prefix="test: " ;;
        security) prefix="security: " ;;
        *) prefix="chore: " ;;
    esac

    local title=""
    if [[ -n "$issue_title" ]]; then
        title=$(echo "$issue_title" | sed 's/\[.*\]:*//g' | sed 's/^ *//')
    else
        title=$(echo "$branch_name" | sed 's/^[^\/]*\///' | sed 's/-/ /g')
    fi

    # Combine and limit to 72 chars
    echo "${prefix}${title}" | cut -c1-72
}
```

### Build Changes Section

```bash
build_changes_section() {
    local base_branch="$1"
    local changes=""

    # Get file status
    local file_changes=$(git diff "$base_branch...HEAD" --name-status)

    # Group by type
    local added=$(echo "$file_changes" | awk '/^A/ {print $2}')
    local modified=$(echo "$file_changes" | awk '/^M/ {print $2}')
    local deleted=$(echo "$file_changes" | awk '/^D/ {print $2}')

    if [[ -n "$added" ]]; then
        changes+="### Added\n"
        while IFS= read -r file; do
            changes+="- [ ] $file\n"
        done <<< "$added"
        changes+="\n"
    fi

    if [[ -n "$modified" ]]; then
        changes+="### Modified\n"
        while IFS= read -r file; do
            changes+="- [ ] $file\n"
        done <<< "$modified"
        changes+="\n"
    fi

    if [[ -n "$deleted" ]]; then
        changes+="### Deleted\n"
        while IFS= read -r file; do
            changes+="- [ ] $file\n"
        done <<< "$deleted"
        changes+="\n"
    fi

    echo -e "$changes"
}
```

## Best Practices

1. **Always Verify Branch**
   - Never create PR from main/master
   - Confirm user is on correct feature branch
   - Check branch has commits ahead of base

2. **Intelligent Auto-Fill**
   - Use git history for deterministic fields
   - Extract context from issue when available
   - Auto-detect PR type from multiple signals
   - Generate meaningful commit-based descriptions

3. **Request Manual Input Judiciously**
   - Only prompt for truly subjective fields
   - Provide sensible defaults where possible
   - Make it easy to skip optional fields
   - Use AskUserQuestion for structured input

4. **Follow Conventions**
   - Use conventional commit prefixes
   - Limit title to 72 characters
   - Reference issues with "Fixes #N" format
   - Maintain template structure

5. **Comprehensive Summary**
   - List what was auto-filled
   - Highlight what needs manual review
   - Provide edit link
   - Include key statistics

## Error Handling

### Not on Feature Branch

```bash
if [[ "$current_branch" == "main" || "$current_branch" == "master" ]]; then
    echo "❌ Cannot create PR from main/master branch"
    echo ""
    echo "To create a PR:"
    echo "1. Create a feature branch: git checkout -b feature/my-feature"
    echo "2. Or use github-issue-implementer skill to create branch from issue"
    echo "3. Make your changes and commit them"
    echo "4. Then run this skill again"
    exit 1
fi
```

### No Commits on Branch

```bash
commit_count=$(git rev-list --count "$base_branch..HEAD")
if [[ "$commit_count" -eq 0 ]]; then
    echo "❌ No commits on branch $current_branch"
    echo "Branch has no changes compared to $base_branch"
    echo ""
    echo "Make some changes and commit them first:"
    echo "  git add <files>"
    echo "  git commit -m 'your message'"
    exit 1
fi
```

### Issue Not Found

```bash
if ! gh issue view "$issue_number" &>/dev/null; then
    echo "⚠ Issue #$issue_number not found or not accessible"
    echo "Proceeding without issue context"
    issue_number=""
    issue_title=""
    issue_body=""
fi
```

### PR Creation Fails

```bash
if ! pr_url=$(gh pr create --title "$pr_title" --body-file "$temp_pr_file" 2>&1); then
    echo "❌ Failed to create PR"
    echo "Error: $pr_url"
    echo ""
    echo "Troubleshooting:"
    echo "1. Check gh auth: gh auth status"
    echo "2. Verify remote exists: git remote -v"
    echo "3. Ensure branch is pushed: git push -u origin $current_branch"
    echo "4. Check you have write access to repository"
    exit 1
fi
```

### Uncommitted Changes Warning

```bash
if [[ -n $(git status --porcelain) ]]; then
    echo "⚠ Warning: You have uncommitted changes"
    echo ""
    echo "Files with changes:"
    git status --short
    echo ""
    # Use AskUserQuestion: "Create PR with uncommitted changes excluded?"
    # Options: "Yes, PR only committed changes", "No, let me commit first"
fi
```

## Advanced Features

### Multi-Commit PR with Grouped Changes

For PRs with many commits, group by commit type:

```bash
# Parse commits by type
features=$(git log "$base_branch..HEAD" --pretty=format:"%s" --grep="^feat:")
fixes=$(git log "$base_branch..HEAD" --pretty=format:"%s" --grep="^fix:")
docs=$(git log "$base_branch..HEAD" --pretty=format:"%s" --grep="^docs:")

# Build enhanced description
if [[ -n "$features" ]]; then
    pr_body+="### Features Added\n"
    pr_body+="$features\n\n"
fi

if [[ -n "$fixes" ]]; then
    pr_body+="### Bugs Fixed\n"
    pr_body+="$fixes\n\n"
fi
```

### Link to CI/CD Status

If the project uses CI/CD:

```bash
# Add CI status section
pr_body+="## CI/CD Status\n\n"
pr_body+="Once PR is created, check:\n"
pr_body+="- [ ] All tests pass\n"
pr_body+="- [ ] Linting passes\n"
pr_body+="- [ ] Build succeeds\n"
pr_body+="- [ ] Coverage maintained\n\n"
```

### Suggest Reviewers

Based on file changes, suggest reviewers:

```bash
# Get files changed
files_changed=$(git diff "$base_branch...HEAD" --name-only)

# Suggest reviewers based on git blame
reviewers=$(git blame --line-porcelain $(echo "$files_changed") 2>/dev/null |
    grep "^author " |
    sort |
    uniq -c |
    sort -rn |
    head -3 |
    sed 's/.*author //')

# Add to PR creation
if [[ -n "$reviewers" ]]; then
    gh pr create --reviewer "$reviewers" ...
fi
```

### Template Customization

Allow users to override template location:

```bash
# Check for project-specific template
if [[ -f ".github/pull_request_template.md" ]]; then
    template_path=".github/pull_request_template.md"
else
    # Use skill's bundled template
    template_path="skills/github-pr-creator/templates/pull_request_template.md"
fi
```

## Integration with Other Skills

### With github-issue-implementer

Perfect workflow:

```bash
# Step 1: Use implementer to create branch and implement
github-issue-implementer #42

# Step 2: After implementation, create PR
github-pr-creator

# The skill will:
# - Detect branch created by implementer
# - Extract issue #42 from branch name
# - Use issue context for PR description
# - Create PR linking back to issue
```

### With github-issue-creator

If issue was created with github-issue-creator:

```bash
# Step 1: Create issue
github-issue-creator (creates #42)

# Step 2: Implement on branch
git checkout -b feature/42-my-feature

# Step 3: Create PR
github-pr-creator
# - Extracts #42
# - Uses full issue context
# - Links PR to issue
```

### With Beads Issue Tracker

Synchronize PR creation with Beads:

```bash
# After PR created, update Beads issue
bd update <issue-id> --notes "PR created: $pr_url"
```

## Workflow Diagram

```text
┌─────────────────────────────────────┐
│ 1. Verify on Feature Branch         │
│    - Not main/master                │
│    - Has commits                    │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 2. Extract/Request Issue Number     │
│    - Parse branch name              │
│    - Ask user if not found          │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 3. Fetch Issue Details (if exists)  │
│    - Title, body, labels            │
│    - Comments for context           │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 4. Analyze Git Changes              │
│    - Commit count & messages        │
│    - File changes (A/M/D)           │
│    - Stats (insertions/deletions)   │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 5. Detect PR Type                   │
│    - From issue labels              │
│    - From branch prefix             │
│    - From commit messages           │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 6. Auto-Fill PR Template            │
│    - Description + issue ref        │
│    - Type of change (checked)       │
│    - Changes made (file list)       │
│    - Default checkboxes             │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 7. Prompt for Manual Fields         │
│    - Testing details                │
│    - Security considerations        │
│    - Cost impact                    │
│    - Breaking changes               │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 8. Generate PR Title                │
│    - Conventional commit format     │
│    - From issue or branch           │
│    - Max 72 characters              │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 9. Create Pull Request              │
│    - Use gh pr create               │
│    - Set title, body, base, head    │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 10. Provide Summary & Next Steps    │
│     - PR URL                        │
│     - Auto-filled sections          │
│     - Manual review needed          │
└─────────────────────────────────────┘
```

## Limitations

- Requires `gh` CLI installed and authenticated
- Must be in git repository with GitHub remote
- Assumes single base branch (main or master)
- Cannot create PRs across forks (use `--head user:branch`)
- Requires write access to repository
- Template must exist in skill's templates directory

## Troubleshooting

### gh CLI Not Authenticated

```bash
# Check status
gh auth status

# Login if needed
gh auth login

# Or use token
export GH_TOKEN=ghp_your_token
```

### Branch Not Pushed to Remote

```bash
# If branch only exists locally
git push -u origin "$current_branch"

# Then create PR
gh pr create ...
```

### PR Already Exists

```bash
# Check if PR exists for branch
if gh pr view &>/dev/null; then
    existing_pr=$(gh pr view --json url -q .url)
    echo "ℹ PR already exists: $existing_pr"
    echo "To update: gh pr edit"
    exit 0
fi
```

### Template Not Found

```bash
# Verify template exists
if [[ ! -f "$template_path" ]]; then
    echo "❌ Template not found: $template_path"
    echo "Using default template structure"
    # Proceed with default template
fi
```

## References

- GitHub CLI: <https://cli.github.com/manual/>
- GitHub Pull Requests: <https://docs.github.com/en/pull-requests>
- Conventional Commits: <https://www.conventionalcommits.org/>
- GitHub PR Templates: <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository>
