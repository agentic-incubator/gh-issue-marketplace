# GitHub Issue Templates

This directory contains example GitHub issue templates that work with the **github-issue-creator** skill.

## Usage

To use these templates in your project:

1. **Copy templates to your repository:**

   ```bash
   # From your project root
   mkdir -p .github/ISSUE_TEMPLATE
   cp path/to/these/templates/*.yml .github/ISSUE_TEMPLATE/
   ```

2. **Customize for your project:**
   - Edit the dropdown options to match your project's components
   - Adjust required fields based on your needs
   - Modify labels to match your labeling scheme

3. **Use with github-issue-creator skill:**
   - The skill will automatically detect these templates
   - It will parse the template structure and create properly formatted issues
   - Missing labels will be created automatically

## Available Templates

### bug_report.yml

Template for reporting bugs with fields for:

- Component selection
- Bug description
- Steps to reproduce
- Expected vs actual behavior
- Environment information
- Configuration details
- Logs and error messages
- Severity level

### feature_request.yml

Template for requesting new features with fields for:

- Feature category
- Problem statement
- Proposed solution
- Alternatives considered
- Use cases
- Priority level
- Acceptance criteria
- Cost impact

### question.yml

Template for asking questions with fields for:

- Question category
- The question itself
- Context and background
- Environment details (optional)
- Documentation value indicator

## GitHub Documentation

For more information about GitHub issue templates:

- [About issue forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates)
- [Syntax for issue forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)

## Customization Tips

1. **Labels:** Make sure the labels referenced in templates exist in your repository, or let the github-issue-creator skill create them automatically.

2. **Dropdown Options:** Customize the dropdown values to match your project structure:

   ```yaml
   - type: dropdown
     id: component
     attributes:
       label: Component
       options:
         - Backend API
         - Frontend UI
         - Database
         - Your Component Here
   ```

3. **Required Fields:** Adjust which fields are required vs optional:

   ```yaml
   validations:
     required: true  # or false
   ```

4. **Default Values:** Set helpful defaults:

   ```yaml
   attributes:
     value: |
       Default text here
   ```
