---
name: github-issue-creator
version: 1.0.0
description: Create GitHub issues from templates with automatic label management and document uploads
author: Chris Phillipson
tags: [github, automation, issue-management, workflow]
---

# GitHub Issue Creator Skill

Automates the creation of GitHub issues from templates with intelligent label management and document attachment.

## When to Use This Skill

Use this skill when:

- Creating a GitHub issue from one of the project's templates
- Need to create and associate labels automatically
- Want to attach supporting documents as issue comments
- Converting existing documentation into a GitHub issue
- Need to follow the project's issue template standards

## Capabilities

This skill will:

1. Analyze available issue templates (bug_report, feature_request, question)
2. Parse template fields and validation requirements
3. Create missing labels with appropriate colors
4. Generate properly formatted issue content
5. Create the GitHub issue with correct metadata
6. Upload supporting documents as comments
7. Link related files and references

## Available Templates

This skill works with standard GitHub issue templates. Example templates are bundled with this skill in the `templates/` directory for reference. Users should copy these to their project's `.github/ISSUE_TEMPLATE/` directory.

The bundled example templates include:

### 1. Bug Report (`bug_report.yml`)

**Labels:** `bug`, `triage`

**Key Fields:**

- Component (dropdown)
- Bug Description
- Steps to Reproduce
- Expected vs Actual Behavior
- Environment Information
- Configuration (TOML)
- Logs and Error Messages
- Severity (Low/Medium/High/Critical)

### 2. Feature Request (`feature_request.yml`)

**Labels:** `enhancement`, `feature-request`

**Key Fields:**

- Feature Category (dropdown)
- Problem Statement
- Proposed Solution
- Alternatives Considered
- Use Cases
- Priority (Low/Medium/High/Critical)
- Acceptance Criteria
- Cost Impact

### 3. Question (`question.yml`)

**Labels:** `question`

**Key Fields:**

- Question Category (dropdown)
- Your Question
- Context
- Environment (optional)
- Documentation value checkbox

## Instructions

When this skill is invoked, follow this workflow:

### Step 1: Understand the Request

Ask the user (if not already clear):

- What type of issue? (bug, feature, question)
- What is the main content source? (file path, description, etc.)
- Are there supporting documents to attach?
- Are there custom labels to add?

### Step 2: Read Template and Content

1. Read the appropriate template from `.github/ISSUE_TEMPLATE/`
2. Read the source content file(s) if provided
3. Parse template fields and requirements

### Step 3: Generate Issue Content

Create a markdown file with the issue content following this structure:

```markdown
---
title: "[Type]: <Title>"
labels: ["label1", "label2", "custom-label"]
assignees: pacphi
---

### Field 1 Name
<content>

### Field 2 Name
<content>

...
```

**Important:** Match template field structure exactly:

- Use exact field names from template
- Respect required fields
- Format dropdowns as plain text values
- Format checkboxes as `- [x]` or `- [ ]`
- Use proper code blocks where appropriate

### Step 4: Create Missing Labels

Before creating the issue:

1. Get existing labels: `gh label list`
2. For each label needed:
   - Check if it exists
   - If not, create with: `gh label create "name" --description "desc" --color "hex"`

**Recommended Label Colors:**

- `bug` - #d73a4a (red)
- `enhancement` - #a2eeef (light blue)
- `feature-request` - #0e8a16 (green)
- `security` - #ee0701 (bright red)
- `question` - #d876e3 (purple)
- `documentation` - #0075ca (blue)
- `P0-critical` - #b60205 (dark red)
- `P1-high` - #d93f0b (orange)
- `P2-medium` - #fbca04 (yellow)
- `P3-low` - #0e8a16 (green)

### Step 5: Create the Issue

Use `gh issue create`:

```bash
gh issue create \
  --title "[Type]: Title Here" \
  --body-file /path/to/issue-content.md \
  --label "label1,label2" \
  --assignee pacphi
```

### Step 6: Upload Supporting Documents

For each supporting document:

1. Read the document content
2. Add as a comment to the issue:

```bash
gh issue comment <issue-number> --body-file /path/to/document.md
```

**Or for inline content:**

```bash
gh issue comment <issue-number> --body "## Document Title

$(cat /path/to/document.md)
"
```

### Step 7: Update Issue References

If the issue body references `.archives/` or other uncommitted paths:

1. Edit the issue to replace file paths with comment references:
   - Change: "See `.archives/file.md`"
   - To: "See implementation details in the comments below"

2. Use `gh issue edit`:

```bash
gh issue edit <issue-number> --body-file /path/to/updated-content.md
```

### Step 8: Provide Summary

Report back to the user:

- Issue URL
- Issue number
- Labels added
- Documents uploaded
- Any additional actions taken

## Example Usage

### Example 1: Feature Request with Implementation Plan

**User Request:**
> Create a feature request issue for the secrets management implementation. Use the P0-C4 content and attach the implementation plan.

**Skill Actions:**

1. Read `feature_request.yml` template
2. Read P0-C4 security issue content
3. Generate issue content matching template fields
4. Create labels: `feature-request`, `security`, `P0-critical`
5. Create issue with `gh issue create`
6. Upload implementation plan as comment
7. Update issue to reference comment instead of `.archives/`
8. Return issue URL to user

### Example 2: Bug Report with Logs

**User Request:**
> File a bug for the Ruby extension timeout. Attach the failure logs.

**Skill Actions:**

1. Read `bug_report.yml` template
2. Ask for missing details (steps to reproduce, expected behavior)
3. Create issue with component="VM Setup/Deployment", severity="High"
4. Upload logs as formatted comment
5. Return issue URL

### Example 3: Question with Context

**User Request:**
> Create a question about mise integration best practices.

**Skill Actions:**

1. Read `question.yml` template
2. Generate question content
3. Create issue with category="Architecture/Design"
4. Return issue URL

## Template Field Mappings

### Bug Report Mapping

```yaml
Required:
  - Component: <dropdown value>
  - Bug Description: <text>
  - Steps to Reproduce: <numbered list>
  - Expected Behavior: <text>
  - Actual Behavior: <text>
  - Environment Information: <formatted list>
  - Severity: <dropdown value>

Optional:
  - Configuration: <code block - TOML>
  - Logs: <code block - shell>
  - Additional Context: <text>
```

### Feature Request Mapping

```yaml
Required:
  - Feature Category: <dropdown value>
  - Problem Statement: <text>
  - Proposed Solution: <text>
  - Use Cases: <numbered list>
  - Priority: <dropdown value>

Optional:
  - Alternatives Considered: <text>
  - Acceptance Criteria: <checklist>
  - Implementation: <checkboxes>
  - Additional Context: <text>
  - Cost Impact: <dropdown value>
```

### Question Mapping

```yaml
Required:
  - Question Category: <dropdown value>
  - Your Question: <text>

Optional:
  - Context: <text>
  - Environment: <formatted list>
  - Documentation: <checkboxes>
```

## Helper Functions

When implementing this skill, use these patterns:

### Check Label Exists

```bash
label_exists() {
    local label_name="$1"
    gh label list --json name --jq ".[].name" | grep -qx "$label_name"
}
```

### Create Label If Missing

```bash
create_label_if_missing() {
    local name="$1"
    local description="$2"
    local color="$3"

    if ! label_exists "$name"; then
        gh label create "$name" --description "$description" --color "$color"
        echo "✓ Created label: $name"
    else
        echo "✓ Label exists: $name"
    fi
}
```

### Upload Document as Comment

```bash
upload_document_comment() {
    local issue_number="$1"
    local file_path="$2"
    local title="${3:-Document}"

    if [[ -f "$file_path" ]]; then
        gh issue comment "$issue_number" --body-file "$file_path"
        echo "✓ Uploaded: $title"
    else
        echo "⚠ File not found: $file_path"
    fi
}
```

### Replace File References in Issue

```bash
# Replace .archives/ references with "see comments below"
update_issue_references() {
    local issue_number="$1"
    local temp_file=$(mktemp)

    # Get current body
    gh issue view "$issue_number" --json body -q .body > "$temp_file"

    # Replace .archives references
    sed -i 's|`.archives/[^`]*`|the comments below|g' "$temp_file"
    sed -i 's|Location: `.archives/.*`|Available as a comment on this issue|g' "$temp_file"

    # Update issue
    gh issue edit "$issue_number" --body-file "$temp_file"
    rm "$temp_file"
}
```

## Best Practices

1. **Always Validate Template Compliance**
   - Check all required fields are present
   - Use correct dropdown values from template
   - Format code blocks with proper language tags

2. **Create Self-Contained Issues**
   - Don't reference `.archives/` or uncommitted files
   - Upload all referenced content as comments
   - Update references to point to comments

3. **Use Semantic Labels**
   - Add priority labels (P0-critical, P1-high, etc.)
   - Add component labels (security, performance, etc.)
   - Keep label set minimal and meaningful

4. **Provide Context**
   - Always include relevant code snippets
   - Link to related issues if applicable
   - Explain "why" not just "what"

5. **Follow Project Conventions**
   - Assign to @pacphi (project owner)
   - Use consistent title formats: `[Type]: Description`
   - Include pre-flight checklist items

## Error Handling

### Label Creation Fails

```bash
if ! gh label create "name" ... 2>/dev/null; then
    echo "⚠ Failed to create label, proceeding without it"
    # Continue with issue creation
fi
```

### Document Upload Fails

```bash
if ! gh issue comment "$issue" --body-file "$file" 2>/dev/null; then
    echo "⚠ Failed to upload document, creating inline instead"
    # Embed content directly in issue body
fi
```

### Issue Creation Fails

```bash
if ! issue_url=$(gh issue create ...); then
    echo "❌ Failed to create issue"
    echo "Troubleshooting:"
    echo "1. Check gh auth status"
    echo "2. Verify repo access"
    echo "3. Check template format"
    exit 1
fi
```

## Advanced Features

### Multi-Document Upload

```bash
# Upload multiple related documents
for doc in implementation-plan.md architecture.md testing.md; do
    if [[ -f ".archives/$doc" ]]; then
        gh issue comment "$issue_number" --body-file ".archives/$doc"
        echo "✓ Uploaded: $doc"
    fi
done
```

### Custom Label Colors by Type

```bash
get_label_color() {
    local label_name="$1"

    case "$label_name" in
        bug|P0-critical) echo "b60205" ;;  # Dark red
        security) echo "ee0701" ;;  # Bright red
        enhancement) echo "a2eeef" ;;  # Light blue
        feature-request) echo "0e8a16" ;;  # Green
        question) echo "d876e3" ;;  # Purple
        documentation) echo "0075ca" ;;  # Blue
        P1-high) echo "d93f0b" ;;  # Orange
        P2-medium) echo "fbca04" ;;  # Yellow
        P3-low) echo "0e8a16" ;;  # Green
        *) echo "ededed" ;;  # Gray (default)
    esac
}
```

### Interactive Mode

If information is missing, ask the user:

```bash
# Example interactive prompt
read -p "Issue type (bug/feature/question): " issue_type
read -p "Issue title: " title
read -p "Priority (P0/P1/P2/P3): " priority
read -p "Supporting documents (comma-separated paths): " docs
```

## Skill Workflow Summary

```text
┌─────────────────────────────────────┐
│ 1. Parse User Request               │
│    - Issue type                     │
│    - Content source(s)              │
│    - Additional documents           │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 2. Read Template & Content          │
│    - Load template YAML             │
│    - Read content files             │
│    - Extract key information        │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 3. Generate Issue Content           │
│    - Match template structure       │
│    - Fill required fields           │
│    - Format properly                │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 4. Create Labels (if missing)       │
│    - Check existing labels          │
│    - Create needed labels           │
│    - Use semantic colors            │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 5. Create GitHub Issue              │
│    - Use gh CLI                     │
│    - Set title, labels, assignees   │
│    - Upload body content            │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 6. Upload Supporting Documents      │
│    - Add as issue comments          │
│    - Format properly                │
│    - Preserve structure             │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 7. Clean Up References              │
│    - Remove .archives/ paths        │
│    - Point to issue comments        │
│    - Make self-contained            │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│ 8. Return Summary                   │
│    - Issue URL                      │
│    - Labels added                   │
│    - Documents uploaded             │
└─────────────────────────────────────┘
```

## Usage Examples

### Example 1: Security Issue with Implementation Plan

**User:**
> Use the github-issue-creator skill. Create a feature request for the secrets management implementation using P0-C4 content and attach the implementation plan.

**Skill Response:**

```text
✓ Read template: feature_request.yml
✓ Read content: P0-C4-api-keys-exposed-in-shell-environment.md
✓ Read document: P0-C4-implementation-plan.md
✓ Created labels: feature-request, security, P0-critical
✓ Created issue: https://github.com/user/repo/issues/24
✓ Uploaded implementation plan as comment
✓ Updated issue to remove .archives references

Issue Summary:

- Type: Feature Request
- Title: [Feature]: Transparent Secrets Management with SOPS + age
- Labels: enhancement, feature-request, security, P0-critical
- Documents: 1 (implementation plan)
```

### Example 2: Bug Report with Logs

**User:**
> Create a bug issue for Ruby extension timeout. Use the failure logs from /tmp/ruby-failure.log.

**Skill Response:**

```text
✓ Read template: bug_report.yml
✓ Read logs: /tmp/ruby-failure.log
✓ Created issue: https://github.com/user/repo/issues/25
✓ Uploaded logs as formatted comment

Missing information requested:

- Steps to reproduce (please provide)
- Expected vs actual behavior (please clarify)

Issue created with partial information. Please update via:
  gh issue edit 25
```

### Example 3: Question with Context

**User:**
> Ask a question about mise best practices for multi-language projects.

**Skill Response:**

```text
✓ Read template: question.yml
✓ Created issue: https://github.com/user/repo/issues/26

Issue Summary:

- Type: Question
- Category: Architecture/Design
- Title: [Question]: Best practices for mise in multi-language projects
```

## Common Patterns

### Pattern 1: Converting Documentation to Issue

```bash
# User has documentation in .archives/
# Skill converts to GitHub issue

1. Read documentation file
2. Extract title, problem, solution
3. Map to appropriate template fields
4. Create issue
5. Upload full doc as comment
6. Update issue to reference comment
```

### Pattern 2: Security Issue with Research

```bash
# User has security finding + research + implementation plan

1. Use feature_request template (security enhancement)
2. Problem Statement ← security finding
3. Proposed Solution ← research recommendations
4. Acceptance Criteria ← implementation checklist
5. Upload implementation plan as comment
6. Upload research as second comment
7. Add labels: security, P0-critical
```

### Pattern 3: Bug with Multiple Logs

```bash
# User has bug + multiple log files

1. Use bug_report template
2. Logs section ← primary log file
3. Upload additional logs as separate comments
4. Title each comment: "## Additional Logs: component-name"
```

## Template-Specific Guidelines

### Bug Reports

**Title Format:** `[Bug]: <Short description>`

**Required Information:**

- Clear reproduction steps (numbered list)
- Exact error messages (code blocks)
- Environment details (OS, versions, region)
- Severity assessment (be honest)

**Tips:**

- Include relevant logs only (not entire dumps)
- Remove secrets from configuration examples
- Use shell code blocks for commands

### Feature Requests

**Title Format:** `[Feature]: <Benefit or capability>`

**Required Information:**

- Problem statement (why needed)
- Proposed solution (how it works)
- Use cases (who benefits, when)
- Acceptance criteria (what defines done)

**Tips:**

- Lead with the problem, not the solution
- Include alternatives considered
- Be specific about cost impact
- Provide implementation examples

### Questions

**Title Format:** `[Question]: <Specific question>`

**Required Information:**

- Clear, specific question
- Context (what you're trying to do)

**Tips:**

- Check documentation first
- Provide examples if helpful
- Indicate if answer should be documented

## Integration with Existing Workflow

This skill integrates with:

- **Beads Issue Tracker**: Can create both GitHub and Beads issues
- **Security Hardening**: Streamlines security issue creation
- **Extension Development**: Quick issue creation for extension features
- **Documentation**: Convert docs to questions/feature requests

## Limitations

- Requires `gh` CLI installed and authenticated
- Must be run in git repository with GitHub remote
- Cannot create issues in private repos without access
- Label colors are suggestions (can be customized)

## Troubleshooting

### gh CLI Not Authenticated

```bash
# Fix with:
gh auth login
# or
export GH_TOKEN=ghp_...
```

### Template Not Found

```bash
# Verify templates exist:
ls .github/ISSUE_TEMPLATE/

# Expected:
# - bug_report.yml
# - feature_request.yml
# - question.yml
```

### Label Creation Permission Denied

```bash
# Requires write access to repository
# Check permissions:
gh auth status

# May need to request access from repo owner
```

## Future Enhancements

Potential improvements:

- [ ] Support for custom templates (not just the three)
- [ ] Automatic milestone assignment
- [ ] Project board integration
- [ ] Issue linking (related issues, duplicates)
- [ ] Bulk issue creation from CSV
- [ ] Issue templates validation
- [ ] Automated follow-up comments

## References

- GitHub CLI: <https://cli.github.com/manual/>
- Issue Templates: <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests>
- YAML Issue Forms: <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms>
