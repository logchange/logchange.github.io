# logchange Reference Documentation

> GitHub repository: [https://github.com/logchange/logchange](https://github.com/logchange/logchange)

Complete reference for logchange commands, configuration options, and YAML format specifications.

## Table of Contents

- [CLI Commands](#cli-commands)
- [Maven Plugin Goals](#maven-plugin-goals)
- [Gradle Plugin Tasks](#gradle-plugin-tasks)
- [YAML Entry Format](#yaml-entry-format)
- [Configuration Reference](#configuration-reference)
- [Template System](#template-system)
- [File Structure](#file-structure)
- [Environment Variables](#environment-variables)

## CLI Commands

### Global Options

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--path` | `-p` | `.` | Project root directory path |
| `--help` | `-h` | | Show help information |
| `--version` | `-v` | | Show version information |

### logchange init

Initialize a project with logchange structure.

```bash
logchange init [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Directory to initialize |
| `--inputDir` | `changelog` | Input directory name |
| `--unreleasedVersionDir` | `unreleased` | Unreleased changes directory |
| `--outputFile` | `CHANGELOG.md` | Output changelog filename |

**Examples:**
```bash
# Initialize current directory
logchange init

# Initialize specific directory
logchange init --path /path/to/project

# Custom directory structure
logchange init --inputDir docs/changelog --outputFile CHANGES.md
```

### logchange add

Create a new changelog entry.

```bash
logchange add [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--inputDir` | `changelog` | Input directory |
| `--unreleasedVersionDir` | `unreleased` | Directory for new entries |
| `--fileName` | generated | Entry filename |
| `--batchMode` | `false` | Non-interactive mode |
| `--empty` | `false` | Create empty entry template |
| `--title` | | Entry title |
| `--author` | | Author info (format: "name; nick; url") |
| `--authors` | | Multiple authors (comma-separated) |
| `--type` | | Change type |
| `--link.name` | | Link name |
| `--link.url` | | Link URL |

**Examples:**
```bash
# Interactive entry creation
logchange add

# Batch mode with predefined values
logchange add --batchMode --title "Add user authentication" --type added

# Empty template
logchange add --empty --fileName 001-new-feature.yml

# Multiple authors
logchange add --authors "John Doe; johndoe; https://github.com/johndoe, Jane Smith; jsmith; https://github.com/jsmith"
```

### logchange example

Generate a pre-filled example entry.

```bash
logchange example [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--inputDir` | `changelog` | Input directory |
| `--unreleasedVersionDir` | `unreleased` | Directory for example |
| `--fileName` | `00000-entry.yml` | Example filename |

### logchange generate

Generate CHANGELOG.md from entries.

```bash
logchange generate [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--inputDir` | `changelog` | Input directory |
| `--outputFile` | `CHANGELOG.md` | Output filename |
| `--configFile` | `logchange-config.yml` | Configuration file |

### logchange lint

Validate YAML entries and configuration.

```bash
logchange lint [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--inputDir` | `changelog` | Input directory |
| `--outputFile` | `CHANGELOG.md` | Output filename (for validation) |
| `--configFile` | `logchange-config.yml` | Configuration file |

### logchange release

Create a release by moving unreleased entries.

```bash
logchange release [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--versionToRelease` | **required** | Version number (e.g., 1.2.0) |
| `--unreleasedVersionDir` | `unreleased` | Unreleased directory |
| `--inputDir` | `changelog` | Input directory |
| `--outputFile` | `CHANGELOG.md` | Output filename |
| `--configFile` | `logchange-config.yml` | Configuration file |
| `--generateChangesXml` | `false` | Generate XML changes file |
| `--xmlOutputFile` | `changes.xml` | XML output filename |

**Examples:**
```bash
# Standard release
logchange release --versionToRelease 1.2.0

# Release with XML generation
logchange release --versionToRelease 1.2.0 --generateChangesXml

# Custom unreleased directory
logchange release --versionToRelease 1.2.0 --unreleasedVersionDir unreleased-1.2.0
```

### logchange aggregate

Aggregate changelogs from multiple projects.

```bash
logchange aggregate [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--aggregateVersion` | **required** | Version to aggregate |
| `--inputDir` | `changelog` | Input directory |
| `--configFile` | `logchange-config.yml` | Configuration file |

**Examples:**
```bash
# Aggregate released version
logchange aggregate --aggregateVersion 2.1.0

# Aggregate unreleased changes
logchange aggregate --aggregateVersion unreleased
```

### logchange archive

Archive old versions to reduce directory clutter.

```bash
logchange archive [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--path, -p` | current directory | Project root directory |
| `--version` | **required** | Archive versions up to (inclusive) |
| `--inputDir` | `changelog` | Input directory |
| `--configFile` | `logchange-config.yml` | Configuration file |

## Maven Plugin Goals

### Configuration

```xml
<plugin>
    <groupId>dev.logchange</groupId>
    <artifactId>logchange-maven-plugin</artifactId>
    <version>1.19.5</version>
    <configuration>
        <inputDir>changelog</inputDir>
        <outputFile>CHANGELOG.md</outputFile>
        <unreleasedVersionDir>unreleased</unreleasedVersionDir>
        <configFile>logchange-config.yml</configFile>
    </configuration>
</plugin>
```

### Goals

| Goal | Description | Command |
|------|-------------|---------|
| `logchange:init` | Initialize project | `mvn logchange:init` |
| `logchange:add` | Add new entry | `mvn logchange:add` |
| `logchange:example` | Create example entry | `mvn logchange:example` |
| `logchange:generate` | Generate changelog | `mvn logchange:generate` |
| `logchange:lint` | Validate entries | `mvn logchange:lint` |
| `logchange:release` | Create release | `mvn logchange:release` |
| `logchange:aggregate` | Aggregate projects | `mvn logchange:aggregate` |
| `logchange:archive` | Archive versions | `mvn logchange:archive` |

### Maven Properties

| Property | Default | Description |
|----------|---------|-------------|
| `empty` | `false` | Create empty entry |
| `fileName` | generated | Entry filename |
| `aggregateVersion` | | Version for aggregation |
| `changesXml` | `false` | Generate changes.xml |
| `outputFileXml` | `changes.xml` | XML output file |
| `version` | | Version for archiving |

**Examples:**
```bash
# Empty entry with custom name
mvn logchange:add -Dempty -DfileName=001-my-feature.yml

# Release with XML generation
mvn logchange:release -DchangesXml -DoutputFileXml=maven-changes.xml

# Aggregate specific version
mvn logchange:aggregate -DaggregateVersion=2.1.0

# Archive up to version
mvn logchange:archive -Dversion=1.5.0
```

## Gradle Plugin Tasks

### Configuration

```groovy
logchange {
    rootPath = "."
    inputDir = "changelog"
    unreleasedVersionDir = "unreleased"
    outputFile = "CHANGELOG.md"
    configFile = "logchange-config.yml"
    generateChangesXml = false
    xmlOutputFile = "changes.xml"
}
```

### Tasks

| Task | Description |
|------|-------------|
| `logchangeInit` | Initialize directory structure |
| `logchangeAdd` | Creates new YML entry file |
| `logchangeExample` | Creates pre-filled example entry |
| `logchangeGenerate` | Generates changelog from entries |
| `logchangeLint` | Validates YML files and config |
| `logchangeRelease` | Creates release by moving unreleased files |
| `logchangeAggregate` | Aggregates multiple project changelogs |
| `logchangeArchive` | Archives released versions |

**Examples:**
```bash
# Basic tasks
./gradlew logchangeInit
./gradlew logchangeAdd
./gradlew logchangeGenerate

# Release with XML
./gradlew logchangeRelease -PgenerateChangesXml=true

# Aggregate version
./gradlew logchangeAggregate -PaggregateVersion=2.1.0
```

## YAML Entry Format

### Complete Field Reference

```yaml
# Required header comment
# This file is used by logchange tool to generate CHANGELOG.md 🌳 🪓 => 🪵
# Visit https://github.com/logchange/logchange and leave a star 🌟

# Required fields
title: "Brief description of the change"
authors:
  - name: "Author Full Name"
    nick: "github-username"
    url: "https://github.com/github-username"
type: "added"  # See entry types below

# Optional fields
modules:
  - "module-name"
  - "another-module"

merge_requests:
  - 123
  - 456

issues:
  - 789
  - 101112

links:
  - name: "External Reference"
    url: "https://example.com/reference"
  - name: "JIRA-123"
    url: "https://company.atlassian.net/browse/JIRA-123"

important_notes:
  - "Important information for users"
  - "Migration steps required"
  - "Breaking change details"

configurations:
  - type: "environment variable"
    action: "add"  # add/update/delete
    key: "CONFIG_KEY"
    default_value: "default_value"
    description: "Configuration description"
    more_info: "Additional details with [links](https://example.com)"
  - type: "application.properties"
    action: "update"
    key: "app.feature.enabled"
    default_value: true
    description: "Enable new feature"
```

### Entry Types

| Type | Description | When to Use |
|------|-------------|-------------|
| `added` | New features | New functionality added |
| `changed` | Changes to existing functionality | Modified behavior |
| `deprecated` | Features marked for removal | Features to be removed in future |
| `removed` | Removed features | Features that were removed |
| `fixed` | Bug fixes | Bug corrections |
| `security` | Security fixes | Vulnerability fixes |
| `dependency_update` | Dependency updates | Library/framework updates |
| `other` | Other changes | Changes not fitting other categories |

### Field Validation Rules

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | String | ✅ | Non-empty string |
| `authors` | Array | ✅ | At least one author with name, nick, url |
| `type` | String | ✅ | Must match configured entry types |
| `modules` | Array | ❌ | Array of strings |
| `merge_requests` | Array | ❌ | Array of integers |
| `issues` | Array | ❌ | Array of integers |
| `links` | Array | ❌ | Objects with name and url fields |
| `important_notes` | Array | ❌ | Array of strings |
| `configurations` | Array | ❌ | Objects with required type, action, key fields |

## Configuration Reference

### Complete logchange-config.yml

```yaml
# This file configures logchange tool 🌳 🪓 => 🪵 
# Visit https://github.com/logchange/logchange and leave a star 🌟

changelog:
  # Optional heading for the changelog
  heading: |
    Custom heading text that appears at the top of CHANGELOG.md
    
    Can contain markdown and multiple lines.

  # Custom entry types (overrides defaults)
  entryTypes:
    - key: "feature"
      order: 1
    - key: "bugfix"
      order: 2
    - key: "improvement" 
      order: 3
    - key: "breaking"
      order: 4

  # Customize display labels
  labels:
    unreleased: "Unreleased"
    important_notes: "Important Notes"
    
    types:
      # Labels for entry type sections
      entryTypesLabels:
        added: "Added"
        changed: "Changed"
        deprecated: "Deprecated"
        removed: "Removed"
        fixed: "Fixed"
        security: "Security"
        dependency_update: "Dependency Updates"
        other: "Other"
      
      # Pluralization for change counts
      number_of_changes:
        singular: "change"
        plural: "changes"
    
    # Configuration section labels
    configuration:
      heading: "Configuration Changes"
      type: "Type"
      actions:
        add: "Added"
        update: "Updated"
        delete: "Deleted"
      with_default_value: "with default value"
      description: "Description"

  # Template customization
  templates:
    # Entry format template
    entry: "${prefix}${title} ${merge_requests} ${issues} ${links} ${authors}"
    
    # Author format template
    author: "([${name}](${url}) @${nick})"
    
    # Version summary templates (generated files)
    version_summary_templates:
      - path: "release-notes.html"
      - path: "version-summary.json"
    
    # Full changelog templates
    changelog_templates:
      - path: "changelog.html"
      - path: "changelog.xml"

# Project aggregation configuration
aggregates:
  projects:
    - name: "project-name"
      url: "https://github.com/org/project/archive/main.tar.gz"
      type: "tar.gz"  # Currently only tar.gz supported
      input_dir: "changelog"
    - name: "another-project"
      url: "https://github.com/org/another/archive/master.tar.gz"
      type: "tar.gz"
      input_dir: "docs/changelog"
```

### Template Variables

#### Entry Template Variables

| Variable | Description | Example Output |
|----------|-------------|----------------|
| `${prefix}` | Project name prefix (aggregation) | `**MyProject** - ` |
| `${title}` | Entry title | `Add user authentication` |
| `${merge_requests}` | Formatted merge request links | `!123 !456` |
| `${issues}` | Formatted issue links | `#789 #101112` |
| `${links}` | Formatted external links | `[JIRA-123](https://...)` |
| `${authors}` | Formatted author information | `([John Doe](https://...) @johndoe)` |

#### Author Template Variables

| Variable | Description |
|----------|-------------|
| `${name}` | Author full name |
| `${nick}` | Author nickname/handle |
| `${url}` | Author profile URL |

## Template System

For a hands-on tutorial with sample Jinja templates and available objects, see the dedicated Templating Guide: [templates.md](templates.md).

## File Structure

### Standard Directory Layout

```
project-root/
├── changelog/
│   ├── logchange-config.yml          # Configuration
│   ├── .templates/                   # Custom templates
│   │   ├── release-notes.html
│   │   └── version-summary.json
│   ├── unreleased/                   # Pending changes
│   │   ├── 001-new-feature.yml
│   │   └── 002-bug-fix.yml
│   ├── unreleased-1.2.0/            # Version-specific unreleased
│   │   └── hotfix-security.yml
│   ├── v1.0.0/                      # Released version
│   │   ├── release-date.txt
│   │   ├── version-summary.md
│   │   ├── feature-auth.yml
│   │   └── bugfix-login.yml
│   ├── v1.1.0/
│   │   └── ...
│   └── archive.md                    # Historical changelog
└── CHANGELOG.md                      # Generated changelog
```

### Generated Files

#### release-date.txt
```
2024-08-30
```

#### version-summary.md
```markdown
## [1.2.0] - 2024-08-30

### Added
- New user authentication system !123 #456 ([John Doe](https://github.com/johndoe) @johndoe)

### Fixed
- Memory leak in connection pooling !124 #457 ([Jane Smith](https://github.com/jsmith) @jsmith)

### Important Notes
- Database migration required: run `migrate-auth-tables.sql`
- Update environment variables: add OAUTH_CLIENT_ID and OAUTH_CLIENT_SECRET
```

## Environment Variables

### CLI Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `LOGCHANGE_PATH` | Default project path | `.` |
| `LOGCHANGE_INPUT_DIR` | Default input directory | `changelog` |
| `LOGCHANGE_OUTPUT_FILE` | Default output file | `CHANGELOG.md` |
| `LOGCHANGE_CONFIG_FILE` | Default config file | `logchange-config.yml` |

### CI/CD Variables

Useful environment variables for CI/CD pipelines:

| Variable | Description | Example |
|----------|-------------|---------|
| `CI_COMMIT_TAG` | Git tag for release | `v1.2.0` |
| `GITHUB_REF_NAME` | GitHub ref name | `1.2.0` |
| `CI_PROJECT_PATH` | Project path | `company/project` |

### Usage in CI/CD

```bash
# Extract version from tag
VERSION=${CI_COMMIT_TAG#v}  # Remove 'v' prefix
logchange release --versionToRelease $VERSION

# Dynamic configuration
export LOGCHANGE_INPUT_DIR="docs/changelog"
export LOGCHANGE_OUTPUT_FILE="CHANGES.md"
logchange generate
```


## Best Practices Summary

### File Organization
- Use descriptive filenames
- Include sequence numbers for ordering
- Group related changes in modules

### YAML Content
- Write clear, concise titles
- Include all relevant authors
- Use appropriate entry types
- Document configuration changes

### Team Workflow
- Validate entries in CI/CD
- Review changelog entries in PRs
- Generate changelogs in release pipeline
- Use version-specific unreleased directories for parallel development

### Configuration
- Customize labels and templates for your project
- Use aggregation for multi-repository projects  
- Archive old versions to keep directories manageable

---

📚 For more information, see:
- [Getting Started Guide](getting-started.md)
- [Usage Guide](usage.md) 
- [FAQ](faq.md)
- [GitHub Repository](https://github.com/logchange/logchange)