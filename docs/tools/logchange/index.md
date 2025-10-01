# logchange Documentation

> GitHub repository: [https://github.com/logchange/logchange](https://github.com/logchange/logchange)

<p align="center">
  <img src="https://user-images.githubusercontent.com/25181517/138590008-f98457b3-602a-4af5-9b28-0c499fe7e378.png" />
</p>

<p align="center">
    <a href="https://github.com/logchange/logchange/graphs/contributors">
        <img src="https://img.shields.io/github/contributors/logchange/logchange" alt="Contributors"/></a>
    <a href="https://github.com/logchange/logchange/pulse">
        <img src="https://img.shields.io/github/commit-activity/m/logchange/logchange" alt="Activity"/></a>
    <a href="https://hub.docker.com/repository/docker/logchange/logchange/">
        <img src="https://img.shields.io/docker/v/logchange/logchange?sort=semver&color=green&label=DockerHub" alt="DockerHub"/></a>
    <a href="https://hub.docker.com/repository/docker/logchange/logchange/">
        <img src="https://img.shields.io/docker/pulls/logchange/logchange" alt="DockerHub Pulls"/></a>
    <a href="https://central.sonatype.com/artifact/dev.logchange/logchange-maven-plugin">
        <img src="https://img.shields.io/maven-central/v/dev.logchange/logchange-maven-plugin.svg?label=Maven%20Central" alt="Maven Central"/></a>
    <a href="https://codecov.io/gh/logchange/logchange">
        <img src="https://codecov.io/gh/logchange/logchange/graph/badge.svg?token=SP3V6ZQ039" alt="codecov"/></a>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/logchange/logchange/refs/heads/main/logchange.gif" alt="logchange in action"/>
</p>

## Welcome to logchange! 🌳

logchange is a powerful tool that revolutionizes changelog management by solving common problems like merge conflicts and forgotten entries. Instead of editing `CHANGELOG.md` directly, you create individual YAML files for each change, and logchange generates your final changelog during release.

### 🚀 Key Benefits

- **🪓 Eliminate merge conflicts** - No more conflicts when multiple developers update the same changelog file
- **🌲 Automated changelog generation** - Generate beautiful changelogs during your release process
- **📜 Perfect release notes** - Create reliable release notes automatically from your changelog entries
- **🔧 Multiple distribution options** - Available as CLI, Maven Plugin, and Gradle Plugin

## 📖 Documentation

### Quick Start
- **[Getting Started Guide](getting-started.md)** - New to logchange? Start here!
    - Installation options (CLI, Maven, Gradle)
    - Project initialization
    - Creating your first changelog entry
    - Basic workflow

### Core Documentation
- **[Usage Guide](usage.md)** - Comprehensive usage instructions
    - Detailed workflow explanations
    - YAML entry format and examples
    - Advanced features and configuration
    - CI/CD integration patterns
    - Templates and customization

- **[FAQ](faq.md)** - Frequently asked questions
    - Common issues and solutions
    - Best practices and recommendations
    - Troubleshooting guide
    - Migration strategies

- **[Reference Documentation](reference.md)** - Complete technical reference
        - All CLI commands and options
    - Maven Plugin goals and configuration
    - Gradle Plugin tasks
    - YAML format specification
    - Configuration options
    - Template system

### Project Information
- **[README](README.md)** - Project overview and basic usage
- **[CHANGELOG](CHANGELOG.md)** - Project changelog (generated with logchange!)

## 🏃 Quick Start

### Choose Your Installation Method

| Method | Best For | Installation |
|--------|----------|--------------|
| **CLI** | Any project type | `brew install logchange/tap/logchange` |
| **Maven** | Java/Maven projects | Add plugin to `pom.xml` |
| **Gradle** | Java/Gradle projects | Add plugin to `build.gradle` |

### 30-Second Setup

1. **Initialize your project:**
```bash
# CLI
logchange init

# Maven
mvn logchange:init

# Gradle
./gradlew logchangeInit
```

2. **Create your first entry:**
   ```bash
   # Interactive mode (recommended)
   logchange add
   mvn logchange:add
   ./gradlew logchangeAdd
   ```

3. **Generate your changelog:**
   ```bash
   logchange generate
   mvn logchange:generate
   ./gradlew logchangeGenerate
   ```

## 🛠️ Installation Options

### CLI (Universal)
Works with any project regardless of technology:

```bash
# Homebrew (macOS/Linux)
brew install logchange/tap/logchange

# Docker
docker pull logchange/logchange

# Direct download
# Visit GitHub Releases for binary downloads
```

### Maven Plugin
Perfect for Java projects using Maven:

```xml
<plugin>
    <groupId>dev.logchange</groupId>
    <artifactId>logchange-maven-plugin</artifactId>
    <version>1.19.5</version>
</plugin>
```

### Gradle Plugin
Ideal for Java projects using Gradle:

```groovy
plugins {
    id 'dev.logchange' version '1.19.5'
}
```

## 📝 Basic Workflow

1. **Development Phase**: Create YAML entries for your changes
   ```yaml
   title: Add user authentication feature
   authors:
     - name: Your Name
       nick: yourhandle
       url: https://github.com/yourhandle
   type: added
   ```

2. **Review Phase**: Validate entries before merging
   ```bash
   logchange lint  # or mvn logchange:lint
   ```

3. **Release Phase**: Generate changelog and create release
   ```bash
   logchange release --versionToRelease 1.2.0
   # or mvn logchange:release
   ```

## 🎯 Key Features

### Entry Types
- `added` - New features
- `changed` - Changes to existing functionality  
- `deprecated` - Features that will be removed
- `removed` - Features that were removed
- `fixed` - Bug fixes
- `security` - Security fixes
- `dependency_update` - Dependency updates
- `other` - Other changes

### Advanced Features
- **Multi-version development** with dedicated unreleased directories
- **Module support** for organizing changes by project components
- **Configuration tracking** for deployment-related changes
- **Template customization** for personalized changelog formats
- **Project aggregation** for multi-repository changelog generation
- **Archive management** for historical changelog data

## 🌟 Example Projects

See logchange in action:
- [logchange itself](https://github.com/logchange/logchange/blob/main/CHANGELOG.md)
- [Hofund project](https://github.com/logchange/hofund/blob/master/CHANGELOG.md)

## 🤝 Contributing

We welcome contributions! Here's how you can help:
- 🐛 [Report bugs](https://github.com/logchange/logchange/issues)
- 💡 [Request features](https://github.com/logchange/logchange/issues)
- 📖 Improve documentation
- 💻 Submit code changes

## 🆘 Need Help?

- 📖 Check our comprehensive [FAQ](faq.md)
- 🔍 Browse the [Reference Documentation](reference.md)
- 💬 [Start a discussion](https://github.com/logchange/logchange/discussions)
- 🐛 [Open an issue](https://github.com/logchange/logchange/issues)

## 📜 Standards and References

logchange follows the principles of [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) with enhancements based on real-world experience across various project types.

The merge conflict problem with traditional `CHANGELOG.md` files has been documented by [GitLab](https://about.gitlab.com/blog/2018/07/03/solving-gitlabs-changelog-conflict-crisis/), and logchange provides an elegant solution to this widespread issue.

---

**🌟 Don't forget to star the [logchange repository](https://github.com/logchange/logchange) if you find it helpful!**

*Made with 🌳 by the logchange team*