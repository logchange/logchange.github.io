# Getting Started with logchange

> GitHub repository: [https://github.com/logchange/logchange](https://github.com/logchange/logchange)

Welcome to logchange! 🌳 This tool helps you maintain clean, conflict-free changelogs by storing individual changes as separate YAML files and generating your `CHANGELOG.md` during release.

## What is logchange?

logchange solves common problems with traditional changelog management:

- **🪓 Merge conflicts**: No more conflicts when multiple developers update the same `CHANGELOG.md` file
- **🌲 Forgotten entries**: Automated reminders to document changes as part of your development workflow
- **📜 Release notes**: Automatic generation of release notes from your changelog entries

Instead of editing `CHANGELOG.md` directly, you create individual YAML files for each change, and logchange generates the final changelog during your release process.

## Quick Start

### Choose Your Installation Method

logchange is available in three flavors:

1. **CLI Binary** - Works with any project, regardless of technology
2. **Maven Plugin** - Perfect for Maven-based Java projects
3. **Gradle Plugin** - Ideal for Gradle-based projects

### Option 1: CLI Installation

**Using Homebrew (macOS/Linux):**
```bash
brew install logchange/tap/logchange
```

**Using Docker:**
```bash
docker pull logchange/logchange
```

**Direct binary download:**
Download the latest release from [GitHub Releases](https://github.com/logchange/logchange/releases)

### Option 2: Maven Plugin

Add to your `pom.xml`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>dev.logchange</groupId>
            <artifactId>logchange-maven-plugin</artifactId>
            <version>1.19.5</version>
        </plugin>
    </plugins>
</build>
```

### Option 3: Gradle Plugin

Add to your `build.gradle`:

```groovy
plugins {
    id 'dev.logchange' version '1.19.5'
}
```

## Initialize Your Project

### CLI
```bash
logchange init
```

### Maven
```bash
mvn logchange:init
```

### Gradle
```bash
./gradlew logchangeInit
```

This creates:
- `changelog/` directory
- `changelog/unreleased/` directory for pending changes
- `changelog/logchange-config.yml` configuration file
- Empty `CHANGELOG.md` file

## Create Your First Entry

### Interactive Mode (Recommended for beginners)

**CLI:**
```bash
logchange add
```

**Maven:**
```bash
mvn logchange:add
```

**Gradle:**
```bash
./gradlew logchangeAdd
```

This launches an interactive wizard that guides you through creating a changelog entry.

### Manual Entry Creation

Create a YAML file in `changelog/unreleased/` (e.g., `001-my-feature.yml`):

```yaml
# This file is used by logchange tool to generate CHANGELOG.md 🌳 🪓 => 🪵
title: Add user authentication feature
authors:
  - name: Your Name
    nick: yourhandle
    url: https://github.com/yourhandle
type: added
merge_requests:
  - 123
issues:
  - 456
```

## Generate Your Changelog

### CLI
```bash
logchange generate
```

### Maven
```bash
mvn logchange:generate
```

### Gradle
```bash
./gradlew logchangeGenerate
```

This creates/updates your `CHANGELOG.md` with all entries from the `changelog/unreleased/` directory.

## Release a Version

When you're ready to release:

### CLI
```bash
logchange release --versionToRelease 1.2.0
```

### Maven
```bash
mvn logchange:release
```

### Gradle
```bash
./gradlew logchangeRelease
```

This will:
1. Move all files from `changelog/unreleased/` to `changelog/v1.2.0/`
2. Create a `release-date.txt` file with the current date
3. Generate an updated `CHANGELOG.md`
4. Create a `version-summary.md` file for release notes

## Entry Types

logchange supports these entry types by default:

- `added` - New features
- `changed` - Changes to existing functionality
- `deprecated` - Features that will be removed
- `removed` - Features that were removed
- `fixed` - Bug fixes
- `security` - Security fixes
- `dependency_update` - Dependency updates
- `other` - Other changes

## Next Steps

- Read the [Usage Guide](usage.md) for detailed instructions
- Check the [FAQ](faq.md) for common questions
- Explore the [Reference](reference.md) for complete configuration options
- Customize outputs with templates: see the [Templating Guide](templates.md)

## Example Projects

See logchange in action:
- [logchange itself](https://github.com/logchange/logchange/blob/main/CHANGELOG.md)
- [Hofund project](https://github.com/logchange/hofund/blob/master/CHANGELOG.md)

---

🌟 **Tip**: Add `changelog/unreleased/` to your pull request template checklist to remind developers to document their changes!