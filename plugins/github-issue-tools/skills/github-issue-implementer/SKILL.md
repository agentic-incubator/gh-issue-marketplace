---
name: github-issue-implementer
version: 1.0.0
description: Read GitHub issues, create feature branches, and implement solutions with automated action planning
author: Chris Phillipson
tags: [github, automation, workflow, implementation, branching]
---

# GitHub Issue Implementer Skill

Automates the end-to-end workflow from GitHub issue to implementation: reads issue content,
creates a feature branch, generates an action plan for approval, and implements the solution.

## When to Use This Skill

Use this skill when:

- You need to implement a GitHub issue that's ready for development
- You want automated branch creation with standardized naming
- You need an action plan generated from issue requirements
- You want to ensure all issue details and comments are considered
- You're starting work on a new feature, bug fix, or enhancement

## Capabilities

This skill will:

1. Read GitHub issue content and all comments
2. Extract requirements, acceptance criteria, and context
3. Create a feature branch with semantic naming
4. Generate a comprehensive action plan
5. Present plan for user approval
6. Implement the solution following the approved plan
7. Track progress with TodoWrite tool

## Branch Naming Convention

The skill uses the following branch naming pattern:

```text
<prefix>/<descriptive-name>
```

**Prefixes:**

- `feature/` - New features or enhancements
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions or improvements
- `chore/` - Maintenance tasks

**Descriptive Name Rules:**

- All lowercase
- Dash-separated words
- Descriptive of the work
- Maximum 50 characters
- No special characters except dashes

**Examples:**

- `feature/secrets-management-with-sops`
- `fix/ruby-extension-timeout`
- `docs/mise-integration-guide`
- `refactor/extension-api-compliance`
- `test/volume-persistence-validation`

## Instructions

When this skill is invoked, follow this workflow:

### Step 1: Parse User Request

Identify the issue to implement:

- GitHub issue number (e.g., "42", "#42")
- GitHub issue URL (e.g., "https://github.com/user/repo/issues/42")
- If not provided, ask the user

### Step 2: Fetch Issue Details

Use `gh` CLI to retrieve complete issue information:

```bash
# Get issue details
gh issue view <issue-number> --json title,body,labels,comments,url

# Parse the JSON response
issue_title=$(gh issue view <issue-number> --json title -q .title)
issue_body=$(gh issue view <issue-number> --json body -q .body)
issue_labels=$(gh issue view <issue-number> --json labels -q '.labels[].name')
issue_comments=$(gh issue view <issue-number> --json comments -q '.comments[].body')
issue_url=$(gh issue view <issue-number> --json url -q .url)
```

### Step 3: Analyze Issue Content

Extract key information:

1. **Determine Branch Prefix:**
   - Check labels: `bug` → `fix/`, `enhancement`/`feature-request` → `feature/`
   - Check title: starts with `[Bug]` → `fix/`, `[Feature]` → `feature/`
   - Check body content for keywords
   - Default to `feature/` if unclear

2. **Extract Requirements:**
   - Problem statement
   - Proposed solution
   - Acceptance criteria
   - Use cases
   - Technical constraints

3. **Read All Comments:**
   - Additional context
   - Implementation suggestions
   - Related issues
   - Design decisions

### Step 4: Generate Branch Name

Create a descriptive branch name:

```bash
# Generate from issue title
# Example: "[Feature]: Transparent Secrets Management with SOPS + age"
# Becomes: "feature/transparent-secrets-management-with-sops"

branch_name=$(echo "$issue_title" |
    sed 's/\[.*\]:*//g' |           # Remove [Type] prefix
    tr '[:upper:]' '[:lower:]' |     # Lowercase
    sed 's/[^a-z0-9 ]//g' |          # Remove special chars
    sed 's/ \+/-/g' |                # Spaces to dashes
    sed 's/^-\+//' |                 # Remove leading dashes
    sed 's/-\+$//' |                 # Remove trailing dashes
    cut -c1-50                       # Limit to 50 chars
)

# Combine with prefix
full_branch_name="${prefix}/${branch_name}"
```

### Step 5: Create Feature Branch

```bash
# Ensure we're on the main branch
git checkout main

# Pull latest changes
git pull origin main

# Create and checkout new branch
git checkout -b "$full_branch_name"

# Confirm branch creation
echo "✓ Created branch: $full_branch_name"
```

### Step 6: Generate Action Plan

Based on the issue content, create a structured action plan:

```markdown
## Action Plan for Issue #<number>

**Issue:** <title>
**Branch:** <branch-name>
**Type:** <feature/fix/docs/etc>

### Requirements Summary

<2-3 sentence summary of what needs to be done>

### Implementation Steps

1. <Step 1 description>
   - <substep>
   - <substep>

2. <Step 2 description>
   - <substep>
   - <substep>

3. <Step 3 description>
   - <substep>

### Acceptance Criteria

- [ ] <Criterion 1>
- [ ] <Criterion 2>
- [ ] <Criterion 3>

### Files to Modify/Create

- `path/to/file1.ext` - <purpose>
- `path/to/file2.ext` - <purpose>

### Testing Strategy

- <Test approach 1>
- <Test approach 2>

### Estimated Complexity

<Low/Medium/High> - <reasoning>

### Dependencies

- <Dependency 1>
- <Dependency 2>

### Additional Notes

<Any important context from issue comments>
```

### Step 7: Present Plan for Approval

Use the AskUserQuestion tool to get approval:

```bash
# Show the plan to the user
# Ask: "Ready to proceed with this implementation plan?"
# Options: "Yes, proceed", "Modify plan", "Cancel"
```

If user requests modifications:

- Ask for specific changes
- Update the plan
- Present revised plan
- Repeat until approved

### Step 8: Implement the Plan (if approved)

Use the TodoWrite tool to track progress:

```bash
# Create todos from action plan steps
TodoWrite with items from implementation steps
```

For each step in the plan:

1. Mark todo as `in_progress`
2. Implement the step
3. Validate the implementation
4. Mark todo as `completed`
5. Move to next step

**Implementation Guidelines:**

- Follow project coding standards (check CLAUDE.md)
- Write tests for new functionality
- Update documentation as needed
- Commit logical chunks of work
- Use meaningful commit messages

### Step 9: Final Validation

Before marking complete:

- Run all relevant tests
- Check linting/formatting
- Verify acceptance criteria met
- Review changes against issue requirements

### Step 10: Provide Summary

Report back to the user:

```markdown
## Implementation Complete

**Issue:** #<number> - <title>
**Branch:** <branch-name>
**Commits:** <number> commits

### Completed Work

- ✓ <Item 1>
- ✓ <Item 2>
- ✓ <Item 3>

### Next Steps

- [ ] Review changes: `git diff main`
- [ ] Run tests: <test command>
- [ ] Create pull request
- [ ] Link PR to issue #<number>

### Pull Request Command

```bash
gh pr create --title "<title>" --body "Fixes #<issue-number>" --base main
```

## Example Usage

### Example 1: Feature Request Implementation

**User Request:**

> Use github-issue-implementer skill for issue #24

**Skill Actions:**

1. Fetch issue #24: "Transparent Secrets Management with SOPS + age"
2. Analyze: Type=feature, Priority=P0-critical
3. Create branch: `feature/transparent-secrets-management-with-sops`
4. Generate action plan with 8 implementation steps
5. Present plan → User approves
6. Create todos for tracking
7. Implement each step:
   - Install SOPS and age
   - Create secrets structure
   - Implement encryption wrapper
   - Update documentation
   - Add validation tests
8. Commit work in logical chunks
9. Provide summary with PR command

**Output:**

```text
✓ Fetched issue #24
✓ Created branch: feature/transparent-secrets-management-with-sops
✓ Generated action plan (8 steps)
✓ User approved plan
✓ Implementation complete (6 commits)

Next: gh pr create --title "feat: Transparent Secrets Management" --body "Fixes #24"
```

### Example 2: Bug Fix with Multiple Comments

**User Request:**

> Implement issue #25 - Ruby extension timeout

**Skill Actions:**

1. Fetch issue #25 with 4 comments
2. Extract:
   - Problem: mise install ruby timing out
   - Root cause (from comments): Network latency in Ruby download
   - Proposed solution: Increase timeout, add retry logic
3. Create branch: `fix/ruby-extension-timeout`
4. Generate action plan:
   - Add configurable timeout to extension
   - Implement retry logic with exponential backoff
   - Add timeout validation in prerequisites
   - Update documentation
5. Present plan → User requests adding logging
6. Update plan to include logging
7. User approves revised plan
8. Implement with todos tracking
9. Provide summary

**Output:**

```text
✓ Fetched issue #25 (4 comments analyzed)
✓ Created branch: fix/ruby-extension-timeout
✓ Generated action plan (revised with logging)
✓ Implementation complete (3 commits)

Next: gh pr create --title "fix: Ruby extension timeout handling" --body "Fixes #25"
```

### Example 3: Documentation Improvement

**User Request:**

> Implement #26 about mise best practices

**Skill Actions:**

1. Fetch issue #26: Question about mise in multi-language projects
2. Determine type: docs (based on question label)
3. Create branch: `docs/mise-multi-language-best-practices`
4. Generate action plan:
   - Research current mise usage in codebase
   - Document per-project version management
   - Add examples for Node.js, Python, Rust, Go
   - Create troubleshooting section
   - Update main CLAUDE.md with reference
5. Present plan → User approves
6. Implement documentation
7. Validate with markdownlint
8. Provide summary

## Helper Functions

### Fetch Issue JSON

```bash
fetch_issue_json() {
    local issue_number="$1"
    gh issue view "$issue_number" --json title,body,labels,comments,url
}
```

### Determine Branch Prefix

```bash
determine_branch_prefix() {
    local labels="$1"
    local title="$2"
    local body="$3"

    # Check labels
    if echo "$labels" | grep -qi "bug"; then
        echo "fix"
    elif echo "$labels" | grep -qi "enhancement\|feature"; then
        echo "feature"
    elif echo "$labels" | grep -qi "documentation"; then
        echo "docs"
    elif echo "$labels" | grep -qi "test"; then
        echo "test"
    elif echo "$labels" | grep -qi "refactor"; then
        echo "refactor"
    # Check title
    elif echo "$title" | grep -qi "^\[bug\]"; then
        echo "fix"
    elif echo "$title" | grep -qi "^\[feature\]"; then
        echo "feature"
    elif echo "$title" | grep -qi "^\[docs\]"; then
        echo "docs"
    # Default
    else
        echo "feature"
    fi
}
```

### Generate Branch Name

```bash
generate_branch_name() {
    local title="$1"
    local prefix="$2"

    local clean_name=$(echo "$title" |
        sed 's/\[.*\]:*//g' |           # Remove [Type] prefix
        tr '[:upper:]' '[:lower:]' |     # Lowercase
        sed 's/[^a-z0-9 ]//g' |          # Remove special chars
        sed 's/ \+/-/g' |                # Spaces to dashes
        sed 's/^-\+//' |                 # Remove leading dashes
        sed 's/-\+$//' |                 # Remove trailing dashes
        cut -c1-50                       # Limit to 50 chars
    )

    echo "${prefix}/${clean_name}"
}
```

### Extract Acceptance Criteria

```bash
extract_acceptance_criteria() {
    local issue_body="$1"

    # Look for acceptance criteria section
    echo "$issue_body" |
        sed -n '/### Acceptance Criteria/,/^###/p' |
        grep '^\- \[' |
        sed 's/^\- \[.\] //'
}
```

### Parse Issue Comments

```bash
parse_comments() {
    local issue_number="$1"

    gh issue view "$issue_number" --json comments \
        -q '.comments[] | "**Comment by @\(.author.login):**\n\(.body)\n"'
}
```

## Best Practices

1. **Always Read All Comments**
   - Comments often contain crucial implementation details
   - Design decisions may be discussed in comments
   - Related issues and dependencies mentioned

2. **Validate Branch Name**
   - Ensure branch name is unique
   - Check it follows conventions
   - Keep it descriptive but concise

3. **Comprehensive Action Plans**
   - Break down into logical steps
   - Include testing strategy
   - Identify file changes needed
   - Note dependencies and risks

4. **Get User Approval**
   - Always present plan before implementing
   - Allow for plan modifications
   - Confirm understanding of requirements

5. **Track Progress**
   - Use TodoWrite for all implementation steps
   - Mark todos as in_progress/completed
   - Keep user informed of progress

6. **Logical Commits**
   - Commit related changes together
   - Use conventional commit messages
   - Reference issue number in commits

## Action Plan Template

Use this template for generating action plans:

```markdown
## Action Plan for Issue #<NUM>

**Title:** <issue-title>
**Branch:** <branch-name>
**Type:** <type>
**Labels:** <label1>, <label2>
**Complexity:** <Low/Medium/High>

### Summary

<2-3 sentences describing what needs to be done>

### Background

<Context from issue body and comments>

### Implementation Steps

1. **<Step Name>**
   - Task 1
   - Task 2
   - Expected outcome: <description>

2. **<Step Name>**
   - Task 1
   - Task 2
   - Expected outcome: <description>

### Acceptance Criteria (from issue)

- [ ] <Criterion 1>
- [ ] <Criterion 2>

### Files to Modify

- `file1.ext` - <why/what>
- `file2.ext` - <why/what>

### Testing Approach

- <Test 1>
- <Test 2>

### Dependencies

- <Dep 1>
- <Dep 2>

### Risks/Unknowns

- <Risk 1>
- <Risk 2>

### Estimated Effort

<X hours/days> - <reasoning>
```

## Error Handling

### Issue Not Found

```bash
if ! gh issue view "$issue_number" &>/dev/null; then
    echo "❌ Issue #$issue_number not found"
    echo "Verify:"
    echo "1. Issue number is correct"
    echo "2. gh CLI is authenticated: gh auth status"
    echo "3. You have access to the repository"
    exit 1
fi
```

### Branch Already Exists

```bash
if git show-ref --verify --quiet "refs/heads/$branch_name"; then
    echo "⚠ Branch '$branch_name' already exists"
    read -p "Use existing branch? (y/n): " use_existing
    if [[ "$use_existing" != "y" ]]; then
        read -p "Enter new branch name: " branch_name
    fi
fi
```

### Dirty Working Directory

```bash
if [[ -n $(git status --porcelain) ]]; then
    echo "⚠ Working directory has uncommitted changes"
    echo "Options:"
    echo "1. git stash - Stash changes"
    echo "2. git commit - Commit changes"
    echo "3. Exit and clean up manually"
    exit 1
fi
```

### User Rejects Plan

```bash
if [[ "$user_approval" == "Cancel" ]]; then
    echo "ℹ Implementation cancelled by user"
    echo "Branch created but no changes made: $branch_name"
    echo "To delete branch: git branch -d $branch_name"
    exit 0
fi
```

## Integration with Other Skills

### With github-issue-creator

```bash
# Create issue → Implement it
1. Use github-issue-creator to create issue
2. Note the issue number from output
3. Use github-issue-implementer with that number
```

### With Beads Issue Tracker

```bash
# Sync GitHub issue to Beads
1. Implement GitHub issue
2. Create corresponding Beads issue for tracking
3. Link GitHub PR to Beads issue
```

### With PR Workflow

```bash
# After implementation
1. Use github-issue-implementer
2. Review changes
3. Create PR with gh pr create
4. Link PR to original issue
```

## Advanced Features

### Multi-Issue Implementation

For issues with dependencies:

```bash
# Implement parent issue first
github-issue-implementer #42

# Then implement dependent issues
github-issue-implementer #43  # depends on #42
github-issue-implementer #44  # depends on #42
```

### Incremental Implementation

For large issues, break into phases:

```bash
# Phase 1: Core functionality
- Implement basic features
- Commit as: "feat: core implementation (issue #42 - phase 1)"

# Phase 2: Additional features
- Implement enhancements
- Commit as: "feat: additional features (issue #42 - phase 2)"

# Phase 3: Documentation and tests
- Add documentation
- Commit as: "docs: add documentation (issue #42 - phase 3)"
```

### Custom Commit Messages

Use conventional commits format:

```text
<type>(<scope>): <subject>

<body>

Refs: #<issue-number>
```

**Types:**

- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `refactor` - Code refactoring
- `test` - Tests
- `chore` - Maintenance

## Workflow Diagram

```text
┌─────────────────────────────────────┐
│ 1. Get Issue Number/URL             │
│    - From user input                │
│    - Validate format                │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 2. Fetch Issue Details              │
│    - Title, body, labels            │
│    - All comments                   │
│    - URL and metadata               │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 3. Analyze Content                  │
│    - Determine branch prefix        │
│    - Extract requirements           │
│    - Parse acceptance criteria      │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 4. Generate Branch Name             │
│    - Clean title                    │
│    - Apply naming rules             │
│    - Combine with prefix            │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 5. Create Branch                    │
│    - Checkout main                  │
│    - Pull latest                    │
│    - Create feature branch          │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 6. Generate Action Plan             │
│    - Implementation steps           │
│    - Acceptance criteria            │
│    - Files to change                │
│    - Testing strategy               │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 7. Get User Approval                │
│    - Present plan                   │
│    - Allow modifications            │
│    - Confirm to proceed             │
└────────────┬────────────────────────┘
             │
         ┌───┴───┐
         │Approved│
         └───┬───┘
             │
┌────────────▼────────────────────────┐
│ 8. Implement Solution               │
│    - Create todos                   │
│    - Execute each step              │
│    - Commit logical chunks          │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 9. Validate & Test                  │
│    - Run tests                      │
│    - Check linting                  │
│    - Verify acceptance criteria     │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 10. Provide Summary                 │
│     - Completed work                │
│     - Next steps                    │
│     - PR creation command           │
└─────────────────────────────────────┘
```

## Limitations

- Requires `gh` CLI installed and authenticated
- Must be in a git repository with GitHub remote
- Cannot implement issues requiring manual decisions without user input
- Branch naming limited to 50 characters for descriptive part
- Requires clean working directory (no uncommitted changes)

## Troubleshooting

### gh CLI Issues

```bash
# Check authentication
gh auth status

# Re-authenticate if needed
gh auth login

# Set default repo if needed
gh repo set-default owner/repo
```

### Git Issues

```bash
# Check git status
git status

# Ensure on correct branch
git branch

# Pull latest changes
git pull origin main
```

### Issue Access

```bash
# Verify issue exists
gh issue view <number>

# Check repository access
gh repo view

# List your issues
gh issue list --assignee @me
```

## References

- GitHub CLI Manual: <https://cli.github.com/manual/>
- Git Branching: <https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging>
- Conventional Commits: <https://www.conventionalcommits.org/>
- GitHub Issues: <https://docs.github.com/en/issues>
