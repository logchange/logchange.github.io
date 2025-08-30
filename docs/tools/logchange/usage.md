# Usage Guide

This comprehensive guide covers detailed usage patterns, advanced features, and best practices for logchange.

## Table of Contents

- [Basic Workflow](#basic-workflow)
- [YAML Entry Format](#yaml-entry-format)
- [Configuration](#configuration)
- [Advanced Features](#advanced-features)
- [Multi-Version Development](#multi-version-development)
- [CI/CD Integration](#cicd-integration)
- [Templates and Customization](#templates-and-customization)
- [Best Practices](#best-practices)

## Basic Workflow

### 1. Development Phase

During development, create changelog entries for each significant change:

```bash
# Interactive entry creation (recommended)
mvn logchange:add

# Or create manually
touch changelog/unreleased/001-new-feature.yml
```

### 2. Review Phase

Validate your entries before merging:

```bash
# Check syntax and structure
mvn logchange:lint
```

### 3. Release Phase

When ready to release:

```bash
# Create release and generate changelog
mvn logchange:release
```

## YAML Entry Format

### Basic Structure

```yaml
# This file is used by logchange tool to generate CHANGELOG.md 🌳 🪓 => 🪵
# Visit https://github.com/logchange/logchange and leave a star 🌟
title: Description of the change
authors:
  - name: Author Name
    nick: github-handle
    url: https://github.com/github-handle
type: added  # added/changed/deprecated/removed/fixed/security/dependency_update/other
merge_requests:
  - 123
issues:
  - 456
```

### Complete Example

```yaml
title: Add user authentication with OAuth2 support
authors:
  - name: John Doe
    nick: johndoe
    url: https://github.com/johndoe
  - name: Jane Smith
    nick: janesmith
    url: https://github.com/janesmith
modules:
  - authentication
  - security
merge_requests:
  - 145
  - 146
issues:
  - 234
  - 235
links:
  - name: OAuth2 Specification
    url: https://oauth.net/2/
  - name: Security Audit Report
    url: https://internal.company.com/audit-123
type: added
important_notes:
  - "Requires database migration: run `migrate-auth-tables.sql`"
  - "Update environment variables: add OAUTH_CLIENT_ID and OAUTH_CLIENT_SECRET"
configurations:
  - type: environment variable
    action: add
    key: OAUTH_CLIENT_ID
    default_value: ""
    description: OAuth2 client identifier
    more_info: "Contact DevOps team for production values"
  - type: application.properties
    action: add
    key: auth.oauth.enabled
    default_value: true
    description: Enable OAuth2 authentication
```

### Field Reference

| Field | Required | Description |
|-------|----------|-------------|
| `title` | ✅ | Brief description of the change |
| `authors` | ✅ | List of contributors |
| `type` | ✅ | Change category |
| `modules` | ❌ | Affected project modules |
| `merge_requests` | ❌ | Related MR/PR numbers |
| `issues` | ❌ | Related issue numbers |
| `links` | ❌ | External references |
| `important_notes` | ❌ | Critical information for users |
| `configurations` | ❌ | Configuration changes |

## Configuration

### logchange-config.yml

Create `changelog/logchange-config.yml` to customize behavior:

```yaml
# This file configures logchange tool 🌳 🪓 => 🪵 
changelog:
  heading: "Custom heading for your changelog"
  
  # Custom entry types (overrides defaults)
  entryTypes:
    - key: feature
      order: 1
    - key: bugfix
      order: 2
    - key: improvement
      order: 3
    - key: breaking
      order: 4
  
  labels:
    unreleased: "🚀 Unreleased"
    important_notes: "⚠️ Important Notes"
    types:
      entryTypesLabels:
        feature: "✨ New Features"
        bugfix: "🐛 Bug Fixes" 
        improvement: "🔧 Improvements"
        breaking: "💥 Breaking Changes"
        security: "🔒 Security"
        dependency_update: "📦 Dependencies"
      number_of_changes:
        singular: "change"
        plural: "changes"
    
    configuration:
      heading: "⚙️ Configuration Changes"
      type: "Type"
      actions:
        add: "➕ Added"
        update: "🔄 Updated" 
        delete: "➖ Removed"
      with_default_value: "with default value"
      description: "Description"
  
  templates:
    entry: "${prefix}${title} ${merge_requests} ${issues} ${links} ${authors}"
    author: "([${name}](${url}) @${nick})"
```

### Template Variables

Available variables for entry templates:

- `${prefix}` - Project name prefix (for aggregated changelogs)
- `${title}` - Entry title
- `${merge_requests}` - Formatted MR/PR links (!123)
- `${issues}` - Formatted issue links (#456)
- `${links}` - External links
- `${authors}` - Formatted author information

## Advanced Features

### Modules

Group related changes by modules:

```yaml
title: Update database connection pooling
modules:
  - database
  - performance
type: changed
```

Changes with the same module are grouped together in the generated changelog.

### Version Summaries

Each release creates a `version-summary.md` file perfect for GitHub releases:

```bash
# After release, find summary at:
cat changelog/v1.2.0/version-summary.md
```

### Custom Templates

Create custom output templates in `changelog/.templates/`:

#### Version Summary Template

`changelog/.templates/release-notes.html`:
```html
<h2>{{ version.version }} Release Notes</h2>
<p>Released: {{ version.releaseDateTime }}</p>

{% for entriesGroup in version.entriesGroups %}
{% if entriesGroup.notEmpty %}
<h3>{{ entriesGroup.type }}</h3>
<ul>
{% for entry in entriesGroup.entries %}
<li>{{ entry }}</li>
{% endfor %}
</ul>
{% endif %}
{% endfor %}
```

Register in `logchange-config.yml`:
```yaml
changelog:
  templates:
    version_summary_templates:
      - path: release-notes.html
```

### Archives

Move old changelog content to archives:

```bash
# Archive versions up to v1.5.0
mvn logchange:archive -Dversion=1.5.0
```

This moves version directories to `changelog/archive.md`.

### Changelog Aggregation

Combine changelogs from multiple projects:

```yaml
# logchange-config.yml
aggregates:
  projects:
    - name: mobile-app-ios
      url: https://github.com/company/mobile-app-ios/archive/main.tar.gz
      type: tar.gz
      input_dir: changelog
    - name: mobile-app-android  
      url: https://github.com/company/mobile-app-android/archive/main.tar.gz
      type: tar.gz
      input_dir: changelog
```

```bash
# Generate aggregated changelog for version 2.1.0
mvn logchange:aggregate -DaggregateVersion=2.1.0
```

## Multi-Version Development

### Dedicated Unreleased Directories

For parallel version development, use version-specific unreleased directories:

```bash
changelog/
├── unreleased-1.3.7/     # Changes for v1.3.7
├── unreleased-1.4.0/     # Changes for v1.4.0
├── unreleased/           # Default unreleased changes
```

When releasing v1.3.7, logchange looks for `unreleased-1.3.7` first, then falls back to `unreleased`.

### Branch Management

**❌ Don't do this:**
```bash
# Two branches with same unreleased directory
git checkout feature-branch-1.1
# Add entries to changelog/unreleased/
git checkout feature-branch-1.2  
# Add entries to changelog/unreleased/
git merge feature-branch-1.1  # ← Creates combined release!
```

**✅ Do this instead:**
```bash
git checkout feature-branch-1.1
# Add entries to changelog/unreleased-1.1/
git checkout feature-branch-1.2
# Add entries to changelog/unreleased-1.2/
```

## CI/CD Integration

### GitHub Actions

`.github/workflows/changelog.yml`:
```yaml
name: Changelog Validation
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '11'
          distribution: 'temurin'
      - name: Validate Changelog
        run: mvn logchange:lint
```

### Release Workflow

`.github/workflows/release.yml`:
```yaml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '11'
          distribution: 'temurin'
      - name: Release Changelog
        run: mvn logchange:release
      - name: Create GitHub Release
        uses: actions/create-release@v1
        with:
          tag_name: ${{ github.ref }}
          release_name: ${{ github.ref }}
          body_path: changelog/v${{ github.ref_name }}/version-summary.md
```

### GitLab CI

`.gitlab-ci.yml`:
```yaml
validate-changelog:
  stage: test
  script:
    - mvn logchange:lint
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

release:
  stage: deploy
  script:
    - mvn logchange:release
    - git add .
    - git commit -m "Release $CI_COMMIT_TAG [skip ci]"
    - git push origin HEAD:$CI_DEFAULT_BRANCH
  rules:
    - if: $CI_COMMIT_TAG
```

## Templates and Customization

### Entry Templates

Customize how entries appear in your changelog:

```yaml
# logchange-config.yml
changelog:
  templates:
    entry: "**${title}** ${authors} ${merge_requests} ${issues}"
    author: "by [${name}](${url})"
```

### Conditional Content

Use different templates based on entry type:

```yaml
templates:
  entry: |
    {% if entry.type == 'security' %}🔒 **Security**: {% endif %}
    ${title} ${authors} ${merge_requests}
```

## Best Practices

### File Naming

Use descriptive, sequential names:
```bash
001-user-authentication.yml
002-password-reset-feature.yml  
003-fix-login-validation.yml
```

Or use semantic naming:
```bash
feature-user-auth.yml
bugfix-login-validation.yml
improvement-password-policy.yml
```

### Entry Content

**Good titles:**
- "Add OAuth2 authentication support"
- "Fix memory leak in connection pooling"
- "Improve error handling in user registration"

**Avoid:**
- "Updates" (too vague)
- "Bug fixes" (not specific)
- "Minor changes" (not descriptive)

### Team Workflow

1. **PR Template**: Include changelog reminder
```markdown
## Checklist
- [ ] Added changelog entry in `changelog/unreleased/`
- [ ] Entry follows naming convention
- [ ] Entry includes proper type and description
```

2. **Code Review**: Verify changelog entries
3. **Release Process**: Use automated releases in CI/CD

### Configuration Management

Keep configuration changes documented:
```yaml
configurations:
  - type: environment variable
    action: add
    key: NEW_FEATURE_ENABLED
    default_value: false
    description: Feature flag for new functionality
    more_info: "Enable in production after testing"
```

### Large Teams

For large teams, consider:
- Module-based organization
- Dedicated changelog maintainers
- Automated entry generation from commit messages
- Regular changelog reviews

---

💡 **Pro tip**: Use `mvn logchange:example` to create a pre-filled example entry that you can modify as a starting point.