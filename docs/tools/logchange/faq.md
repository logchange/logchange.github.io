# Frequently Asked Questions (FAQ)

> GitHub repository: [https://github.com/logchange/logchange](https://github.com/logchange/logchange)

This FAQ addresses common questions, issues, and use cases when working with logchange.

## Table of Contents

- [Getting Started](#getting-started)
- [YAML Entries](#yaml-entries)
- [Configuration](#configuration)
- [Workflow and Process](#workflow-and-process)
- [CI/CD Integration](#cicd-integration)
- [Troubleshooting](#troubleshooting)
- [Migration](#migration)
- [Advanced Usage](#advanced-usage)

## Getting Started

### Q: Which distribution should I choose (CLI, Maven, or Gradle)?

**A:** Choose based on your project type:
- **CLI**: Universal solution, works with any project (Node.js, Python, Go, etc.)
- **Maven Plugin**: Best for Java/JVM projects using Maven
- **Gradle Plugin**: Best for Java/JVM projects using Gradle

The CLI version has all the same features as the plugins but requires manual version specification during releases.

### Q: Can I use logchange in an existing project with an existing CHANGELOG.md?

**A:** Yes! Move your existing `CHANGELOG.md` to `changelog/archive.md` before running `logchange init`. logchange will preserve your historical changelog and append new entries.

### Q: Do I need to commit the generated CHANGELOG.md?

**A:** It depends on your workflow:
- **Recommended**: Generate in CI/CD and commit automatically during releases
- **Alternative**: Commit manually after each release
- **Avoid**: Committing on every change (defeats the purpose of avoiding merge conflicts)

## YAML Entries

### Q: What's the minimum required content for a YAML entry?

**A:** The minimal entry requires:
```yaml
title: Description of the change
authors:
  - name: Your Name
    nick: yourhandle  
    url: https://github.com/yourhandle
type: added
```

### Q: Can I have multiple authors for a single change?

**A:** Yes, list multiple authors:
```yaml
authors:
  - name: John Doe
    nick: johndoe
    url: https://github.com/johndoe
  - name: Jane Smith
    nick: janesmith
    url: https://github.com/janesmith
```

### Q: How do I handle breaking changes?

**A:** Use the `important_notes` field and appropriate entry type:
```yaml
title: Rename API endpoint from /users to /accounts
type: changed
important_notes:
  - "BREAKING: Update API calls from /users to /accounts"
  - "Migration script available: scripts/migrate-api-calls.sh"
```

### Q: Should I create entries for dependency updates?

**A:** For major updates or security patches, yes:
```yaml
title: Upgraded Spring Boot from 2.7 to 3.0
type: dependency_update
important_notes:
  - "Review application for Spring Boot 3.0 compatibility changes"
```

For minor/patch updates, it's optional and depends on your team's preferences.

### Q: How do I reference issues from different systems (JIRA, Linear, etc.)?

**A:** Use the `links` field for external systems:
```yaml
issues:
  - 123  # GitHub/GitLab issue
links:
  - name: PROJ-456
    url: https://company.atlassian.net/browse/PROJ-456
  - name: Linear Issue
    url: https://linear.app/company/issue/ABC-123
```

## Configuration

### Q: Can I customize the entry types?

**A:** Yes, define custom entry types in `logchange-config.yml`:
```yaml
changelog:
  entryTypes:
    - key: feature
      order: 1
    - key: bugfix  
      order: 2
    - key: breaking
      order: 3
```

### Q: How do I change the changelog heading/description?

**A:** Set the `heading` in your configuration:
```yaml
changelog:
  heading: |
    This changelog documents all notable changes to MyProject.
    
    The format is based on [Keep a Changelog](https://keepachangelog.com/).
```

### Q: Can I use emojis in my changelog?

**A:** Absolutely! Customize labels in your configuration:
```yaml
changelog:
  labels:
    types:
      entryTypesLabels:
        added: "✨ New Features"
        fixed: "🐛 Bug Fixes"
        security: "🔒 Security"
```

## Workflow and Process

### Q: When should I create changelog entries?

**A:** Create entries when:
- Adding new features
- Fixing bugs that affect users
- Making breaking changes
- Updating dependencies (major versions)
- Changing configuration requirements

**Don't create entries for:**
- Internal refactoring without user impact
- Test updates
- Documentation-only changes
- Minor dependency patches

### Q: How do I handle hotfixes?

**A:** Create entries in the appropriate unreleased directory:
```bash
# For hotfix to v1.2.x
changelog/unreleased-1.2.3/hotfix-security-issue.yml

# For next major release  
changelog/unreleased/feature-new-api.yml
```

### Q: What's the best file naming convention?

**A:** Use descriptive, consistent names:

**Sequential numbering:**
```
001-user-authentication.yml
002-password-reset.yml
003-fix-login-bug.yml
```

**Semantic naming:**
```
feature-oauth-integration.yml
bugfix-memory-leak.yml
security-sql-injection-fix.yml
```

**Mixed approach:**
```
001-feature-oauth-integration.yml
002-bugfix-memory-leak.yml  
003-security-sql-injection-fix.yml
```

### Q: How do I manage changes across multiple branches?

**A:** Use dedicated unreleased directories:
- `unreleased/` - Default branch changes
- `unreleased-1.5.0/` - v1.5.0 branch changes  
- `unreleased-2.0.0/` - v2.0.0 branch changes

This prevents conflicts when merging branches with different target versions.

## CI/CD Integration

### Q: How do I validate entries in pull requests?

**A:** Add a lint step to your CI:
```yaml
# GitHub Actions
- name: Validate Changelog Entries
  run: mvn logchange:lint
```

### Q: Can I auto-generate entries from commit messages?

**A:** Not directly, but you can:
1. Use commit message conventions (Conventional Commits)
2. Write a script to generate YAML from commits
3. Review and edit generated entries before release

### Q: How do I prevent releases without changelog entries?

**A:** Check for unreleased entries in your CI:
```bash
# Fail if no unreleased entries exist
if [ ! "$(ls -A changelog/unreleased/)" ]; then
  echo "No changelog entries found for release!"
  exit 1
fi
```

### Q: How do I automatically create GitHub releases with changelog content?

**A:** Use the generated `version-summary.md`:
```yaml
- name: Create GitHub Release
  uses: actions/create-release@v1
  with:
    tag_name: ${{ github.ref }}
    body_path: changelog/v${{ steps.version.outputs.version }}/version-summary.md
```

## Troubleshooting

### Q: I'm getting "YAML parse error" when running lint

**A:** Common YAML issues:
- **Indentation**: Use spaces, not tabs
- **Quotes**: Use quotes for values containing special characters
- **Lists**: Ensure proper list syntax with `-`

```yaml
# ❌ Wrong
authors:
- name:John Doe  # Missing space after colon
```

```yaml
# ✅ Correct  
authors:
  - name: John Doe
```

### Q: My changes aren't appearing in the generated changelog

**A:** Check:
1. Files are in the correct directory (`changelog/unreleased/`)
2. Files have `.yml` or `.yaml` extension
3. YAML syntax is valid (`mvn logchange:lint`)
4. You've run the generate command after adding entries

### Q: The release command isn't finding my version

**A:** For Maven plugin, ensure:
- Version is set in `pom.xml`
- You're running from the project root
- For CLI, specify version: `--versionToRelease 1.2.0`

### Q: I accidentally released the wrong version

**A:** You can manually fix this:
1. Move the version directory: `mv changelog/v1.2.0 changelog/v1.1.5`
2. Update `release-date.txt` if needed
3. Regenerate changelog: `mvn logchange:generate`

### Q: How do I fix merge conflicts in changelog entries?

**A:** Conflicts in YAML files are rare but can happen:
1. Review both versions of the conflicted entry
2. Merge the content manually  
3. Ensure valid YAML syntax
4. Run `mvn logchange:lint` to validate

## Migration

### Q: How do I migrate from a traditional CHANGELOG.md?

**A:** 
1. **Backup** your existing `CHANGELOG.md`
2. **Move** it to `changelog/archive.md`
3. **Initialize** logchange: `mvn logchange:init`
4. **Start** creating YAML entries for new changes

### Q: Can I migrate my Git history to create historical entries?

**A:** While possible, it's not recommended for large histories. Instead:
- Keep historical changelog as `archive.md`
- Start fresh with logchange for new changes
- Optionally create entries for recent unreleased changes

### Q: How do I convert existing unreleased changes to YAML?

**A:** Manually create YAML entries for each change:
```yaml
title: Feature description from old changelog
authors:
  - name: Author Name
    nick: handle
    url: https://github.com/handle
type: added  # Determine appropriate type
```

## Advanced Usage

### Q: How do I aggregate changelogs from multiple repositories?

**A:** Configure aggregation in `logchange-config.yml`:
```yaml
aggregates:
  projects:
    - name: frontend
      url: https://github.com/company/frontend/archive/main.tar.gz
      type: tar.gz
      input_dir: changelog
    - name: backend
      url: https://github.com/company/backend/archive/main.tar.gz  
      type: tar.gz
      input_dir: changelog
```

Then run: `mvn logchange:aggregate -DaggregateVersion=2.1.0`

### Q: Can I customize the output format beyond markdown?

**A:** Yes, create custom templates:
- **Version summaries**: HTML, JSON, XML templates
- **Full changelog**: Custom markdown templates
- **Integration**: Generate data for other tools

See the [Templating Guide](templates.md) for examples, available objects, and best practices.

### Q: How do I handle configuration changes that need documentation?

**A:** Use the `configurations` field:
```yaml
title: Add Redis caching support
type: added
configurations:
  - type: environment variable
    action: add
    key: REDIS_URL
    default_value: redis://localhost:6379
    description: Redis server connection URL
    more_info: "Required for production deployment"
```

### Q: Can I have project-specific entry types?

**A:** Yes, define them in your `logchange-config.yml`:
```yaml
changelog:
  entryTypes:
    - key: api_change
      order: 1
    - key: database_migration  
      order: 2
    - key: performance_improvement
      order: 3
```

### Q: How do I handle entries that affect multiple modules?

**A:** List all affected modules:
```yaml
title: Update authentication across services
modules:
  - user-service
  - admin-panel
  - mobile-api
type: security
```

The entry will appear under each module section in the generated changelog.

---

## Still have questions?

- 📖 Check the [Usage Guide](usage.md) for detailed instructions
- 📚 Review the [Reference Documentation](reference.md) for complete options
- 🐛 [Open an issue](https://github.com/logchange/logchange/issues) for bugs or feature requests
- 💬 [Start a discussion](https://github.com/logchange/logchange/discussions) for general questions

**🌟 Don't forget to star the [logchange repository](https://github.com/logchange/logchange) if you find it helpful!**