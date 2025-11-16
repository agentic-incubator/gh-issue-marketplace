# GitHub Issue Marketplace

A Claude Code marketplace providing intelligent automation for GitHub issue workflows.

## Overview

This marketplace contains the **github-issue-tools** plugin, which provides two powerful skills for automating GitHub issue management:

- **github-issue-creator**: Automates creation of GitHub issues from templates with intelligent label management and document uploads
  - Includes example GitHub issue templates (bug report, feature request, question)
- **github-issue-implementer**: Reads GitHub issues, creates feature branches, generates action plans, and implements solutions

## Prerequisites

Before using this marketplace, ensure you have:

1. **Claude Code CLI** installed and configured
   - Install from:

      ```url
      https://code.claude.com
      ```

   - Verify installation: `claude --version`

1. **GitHub CLI (gh)** installed and authenticated

   ```bash
   # Install gh (macOS)
   brew install gh

   # Or install on other platforms
   # See: https://cli.github.com/

   # Authenticate with GitHub
   gh auth login

   # Verify authentication
   gh auth status
   ```

2. **Git** configured with your credentials

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

## Quick Start

### 1. Clone the Marketplace

```bash
# Clone this repository
git clone https://github.com/agentic-incubator/gh-issue-marketplace.git

# Navigate to the repository
cd gh-issue-marketplace
```

### 2. Add the Marketplace to Claude Code

You can add the marketplace in two ways:

#### Option A: Add via GitHub Repository (Recommended)

```bash
# Add the marketplace using the repository name
claude plugin marketplace add agentic-incubator/gh-issue-marketplace
```

#### Option B: Add via Local Path

```bash
# Add the marketplace from your local clone
claude plugin marketplace add /path/to/gh-issue-marketplace
```

### 3. Install the Plugin

```bash
# List available plugins in the marketplace
claude plugin marketplace list

# Install the github-issue-tools plugin
claude plugin install github-issue-tools
```

### 4. Verify Installation

```bash
# List installed plugins
claude plugin list

# You should see 'github-issue-tools' in the output
```

## Using the Skills

Once installed, the skills are automatically available in Claude Code. Claude will intelligently invoke them based on your requests.

### Skill 1: GitHub Issue Creator

This skill automates creating GitHub issues from templates with proper formatting, labels, and attachments.

#### When to Use

- Creating a GitHub issue from a template
- Need to create and associate labels automatically
- Want to attach supporting documents as issue comments
- Converting existing documentation into a GitHub issue

#### Example Usage

Start a Claude Code session in your GitHub repository:

```bash
# Navigate to your GitHub repository
cd /path/to/your/repo

# Start Claude Code
claude
```

Then ask Claude to create an issue:

```text
Create a feature request issue for implementing SOPS-based secrets management.
Use the security hardening document at .archives/P0-C4-api-keys.md and attach
the implementation plan.
```

Claude will:

1. Read the appropriate issue template
2. Parse the content from your document
3. Create any missing labels with appropriate colors
4. Create the GitHub issue with proper formatting
5. Upload supporting documents as comments
6. Return the issue URL

#### Supported Issue Templates

The skill works with standard GitHub issue templates. **Example templates are bundled with the skill** in the `templates/` directory that you can copy to your project:

- `bug_report.yml` - For bug reports
- `feature_request.yml` - For feature requests
- `question.yml` - For questions

To use these templates in your project, copy them to `.github/ISSUE_TEMPLATE/` in your repository. See the templates directory README for detailed instructions.

### Skill 2: GitHub Issue Implementer

This skill automates the workflow from issue to implementation: reads issue content, creates a feature branch, generates an action plan, and implements the solution.

#### When to Use

- Implementing a GitHub issue that's ready for development
- Need automated branch creation with standardized naming
- Want an action plan generated from issue requirements
- Starting work on a new feature, bug fix, or enhancement

#### Example Usage

In a Claude Code session:

```
Implement GitHub issue #42
```

Claude will:

1. Fetch the complete issue details including all comments
2. Analyze the requirements and acceptance criteria
3. Create a feature branch with semantic naming (e.g., `feature/secrets-management`)
4. Generate a comprehensive action plan
5. Present the plan for your approval
6. Implement the solution step-by-step (if approved)
7. Track progress with todos
8. Provide a summary with PR creation command

#### Branch Naming Convention

The skill uses semantic prefixes:

- `feature/` - New features or enhancements
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions or improvements
- `chore/` - Maintenance tasks

Branch names are automatically generated from issue titles:

- Lowercase with dashes
- Maximum 50 characters for the descriptive part
- No special characters except dashes

Example:

- Issue: "#42: [Feature] Transparent Secrets Management with SOPS"
- Branch: `feature/transparent-secrets-management-with-sops`

## Marketplace Structure

```text
gh-issue-marketplace/
├── .claude-plugin/
│   └── marketplace.json                     # Marketplace configuration
├── plugins/
│   └── github-issue-tools/                  # Plugin directory
│       ├── plugin.json                      # Plugin manifest
│       └── skills/                          # Skills directory
│           ├── github-issue-creator/
│           │   ├── SKILL.md                 # Issue creator skill
│           │   └── templates/               # Example issue templates
│           │       ├── README.md            # Template usage guide
│           │       ├── bug_report.yml       # Bug report template
│           │       ├── feature_request.yml  # Feature request template
│           │       └── question.yml         # Question template
│           └── github-issue-implementer/
│               └── SKILL.md                 # Issue implementer skill
└── README.md                                # This file
```

## Advanced Usage

### Creating Issues with Custom Labels

```text
Create a bug report for the Ruby extension timeout issue.
Add labels: bug, P1-high, and ruby-extension
```

### Implementing Multiple Related Issues

```text
Implement issues #42, #43, and #44 in sequence
```

### Customizing Action Plans

When Claude presents an action plan, you can request modifications:

```text
Modify the plan to include integration tests and update the documentation
```

## Configuration

### Issue Templates

The skills work best when your repository has GitHub issue templates in `.github/ISSUE_TEMPLATE/`.

**Example templates are bundled with this plugin** at:

```text
plugins/github-issue-tools/skills/github-issue-creator/templates/
```

To use them in your project:

```bash
# Copy templates to your project
cp -r plugins/github-issue-tools/skills/github-issue-creator/templates/*.yml \
     your-project/.github/ISSUE_TEMPLATE/
```

Included templates:

- `bug_report.yml` - Bug reports with severity levels
- `feature_request.yml` - Feature requests with acceptance criteria
- `question.yml` - Questions with categorization

Example bug report template structure:

```yaml
name: Bug Report
description: File a bug report
labels: ["bug", "triage"]
body:
  - type: input
    id: component
    attributes:
      label: Component
      description: Which component is affected?
    validations:
      required: true
  - type: textarea
    id: description
    attributes:
      label: Bug Description
      description: Describe the bug
    validations:
      required: true
  # ... more fields
```

### Label Colors

The issue creator skill uses semantic colors:

- `bug` - #d73a4a (red)
- `enhancement` - #a2eeef (light blue)
- `feature-request` - #0e8a16 (green)
- `security` - #ee0701 (bright red)
- `question` - #d876e3 (purple)
- `documentation` - #0075ca (blue)

Priority labels:

- `P0-critical` - #b60205 (dark red)
- `P1-high` - #d93f0b (orange)
- `P2-medium` - #fbca04 (yellow)
- `P3-low` - #0e8a16 (green)

## Troubleshooting

### Marketplace Not Found

```bash
# Verify the marketplace is added
claude plugin marketplace list

# Re-add if needed
claude plugin marketplace add agentic-incubator/gh-issue-marketplace
```

### Plugin Installation Fails

```bash
# Remove and reinstall
claude plugin uninstall github-issue-tools
claude plugin install github-issue-tools

# Check for errors
claude plugin validate github-issue-tools
```

### GitHub CLI Not Authenticated

```bash
# Check authentication status
gh auth status

# Re-authenticate if needed
gh auth login

# Set default repository (in your repo)
gh repo set-default owner/repo
```

### Skills Not Activating

The skills are model-invoked, meaning Claude decides when to use them based on context. To explicitly invoke a skill:

```text
Use the github-issue-creator skill to create a feature request...
```

Or:

```text
Use the github-issue-implementer skill for issue #42
```

### Permission Errors

Ensure you have:

- Write access to the GitHub repository
- Authenticated `gh` CLI with appropriate scopes
- Git configured with your credentials

## Examples

### Example 1: End-to-End Workflow

```bash
# Start Claude Code in your repo
cd ~/projects/my-app
claude
```

In Claude Code:

```text
Create a feature request for adding dark mode support.
Include acceptance criteria for theme switching, persistence, and accessibility.
Add labels: enhancement, feature-request, and P2-medium.
```

Then:

```text
Implement the issue you just created
```

### Example 2: Bug Report with Logs

```text
Create a bug report for the API timeout issue.
Component: Backend API
Severity: High
Attach the error logs from logs/api-error.log
```

### Example 3: Documentation Issue

```text
Create a question issue asking about best practices for database migrations.
Category: Architecture/Design
Mark it for documentation.
```

## Best Practices

1. **Issue Templates**: Set up comprehensive issue templates in your repository
2. **Clear Descriptions**: Provide detailed information when creating issues
3. **Label Consistency**: Use consistent label naming across your repositories
4. **Branch Hygiene**: Review generated branch names and action plans before proceeding
5. **Commit Messages**: Use conventional commit format (feat:, fix:, docs:, etc.)
6. **Testing**: Always run tests before creating pull requests

## Contributing

To contribute to this marketplace:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with `claude plugin validate`
5. Submit a pull request

## License

Apache-2.0

## Support

For issues or questions:

- GitHub Issues: <https://github.com/agentic-incubator/gh-issue-marketplace/issues>
- Claude Code Docs: <https://code.claude.com/docs>

## References

- [Claude Code Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Claude Code Skills](https://code.claude.com/docs/en/skills)
- [GitHub CLI Manual](https://cli.github.com/manual/)
- [GitHub Issue Templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests)
