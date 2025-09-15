# ❓ Frequently Asked Questions

> GitHub repository: [https://github.com/logchange/valhalla](https://github.com/logchange/valhalla)

This FAQ covers common questions about Valhalla, troubleshooting tips, and clarifications about features and limitations.

## Table of Contents

- [General Questions](#general-questions)
- [Setup and Configuration](#setup-and-configuration)
- [Release Process](#release-process)
- [Variables and Templating](#variables-and-templating)
- [CI/CD Integration](#cicd-integration)
- [Troubleshooting](#troubleshooting)
- [Advanced Usage](#advanced-usage)
- [Limitations and Known Issues](#limitations-and-known-issues)

## General Questions

### What is Valhalla and what problem does it solve?

Valhalla is an automated release management tool designed to eliminate the complexity and error-prone nature of manual software releases. It solves several key problems:

- **Manual release complexity**: Automates the multi-step release process
- **Human errors**: Reduces mistakes through automation and standardization  
- **Time inefficiency**: Eliminates waiting for manual interventions
- **Compliance issues**: Ensures consistent adherence to release standards
- **Multi-repository management**: Supports configuration inheritance for managing many projects

### Which platforms does Valhalla support?

Currently, Valhalla supports:
- ✅ **GitLab** (self-hosted and GitLab.com)
- ❌ **GitHub** (not supported yet)
- ❌ **Bitbucket** (not supported yet)
- ❌ **Azure DevOps** (not supported yet)

### Is Valhalla free to use?

Yes! Valhalla is open-source software available under an open-source license. You can use it freely for both personal and commercial projects.

### Can I use Valhalla with private repositories?

Yes, Valhalla works with both public and private repositories. You just need to provide a GitLab access token with appropriate permissions.

## Setup and Configuration

### What permissions does the GitLab token need?

Your GitLab access token requires these scopes:
- `api` - Full API access for creating releases and merge requests
- `write_repository` - Write access for committing and pushing changes

### Where should I store the GitLab token?

**Never store tokens in your code or configuration files!** Instead:
- Use GitLab CI/CD variables (recommended)
- Use environment variables in your deployment environment
- Use secure secret management systems

```bash
# In GitLab CI/CD Variables
VALHALLA_TOKEN=glpat-xxxxxxxxxxxxxxxxxxxx
```

### Can I test Valhalla without making actual releases?

Yes! There are several ways to test:

1. **Use a test repository** with dummy releases
2. **Implement dry-run logic** in your configuration:
   ```yaml
   variables:
     DRY_RUN: "true"
   
   commit_before_release:
     before:
       - if [ "{DRY_RUN}" = "true" ]; then echo "DRY RUN: Skipping actual changes"; exit 0; fi
   ```
3. **Test locally** with the Python version before deploying to CI/CD

### How do I handle multiple environments (dev, staging, prod)?

You can create different configuration files for different environments:

```yaml
# valhalla-staging.yml
extends:
  - https://raw.githubusercontent.com/company/configs/main/valhalla-base.yml

variables:
  ENVIRONMENT: "staging"
  DEPLOY_TARGET: "staging.company.com"

# Use with: release-staging-1.2.3
```

### What if my repository doesn't have a valhalla.yml file?

Valhalla requires a configuration file. If missing, you'll get an error. Create a minimal configuration:

```yaml
git_host: gitlab
release:
  description:
    from_command: "echo 'Release {VERSION}'"
```

## Release Process

### How does Valhalla detect what version to release?

Valhalla uses two methods (in order of priority):

1. **Environment variable**: `VALHALLA_RELEASE_CMD=release-1.2.3`
2. **Branch name**: Create branches like `release-1.2.3`

The version is extracted from the string after `release-`.

### Can I use custom version formats?

Yes! Valhalla supports various version formats:
- `release-1.2.3` → `1.2.3`
- `release-2.0.0-RC1` → `2.0.0-RC1`
- `release-1.2.3-beta.1` → `1.2.3-beta.1`
- `release-hotfix-1.2.4` → `1.2.4` (uses hotfix config)

### What happens if the release process fails?

If Valhalla fails:
1. **Check the logs** for error details
2. **Fix the issue** (configuration, permissions, etc.)
3. **Re-trigger** the release (push to branch again or retrigger pipeline)
4. **Clean up** any partial changes if necessary

The process is idempotent, so you can safely retry failed releases.

### Can I skip certain steps in the release process?

Yes! You can disable individual steps:

```yaml
commit_before_release:
  enabled: false  # Skip pre-release commits

commit_after_release:
  enabled: false  # Skip post-release commits

merge_request:
  enabled: false  # Skip merge request creation
```

### How do I handle release rollbacks?

Valhalla doesn't provide automatic rollbacks, but you can:

1. **Create a rollback release**: `release-1.2.2-rollback`
2. **Use GitLab's release management** to mark releases as deprecated
3. **Implement rollback scripts** in your configuration:
   ```yaml
   commit_before_release:
     before:
       - ./scripts/rollback-check.sh {VERSION}
   ```

## Variables and Templating

### What variables are available in Valhalla?

Valhalla provides several types of variables:

**Predefined variables** (highest priority):
- `{VERSION}` - Full version (e.g., "1.2.3")
- `{VERSION_MAJOR}` - Major version (e.g., "1")
- `{VERSION_MINOR}` - Minor version (e.g., "2")
- `{VERSION_PATCH}` - Patch version (e.g., "3")
- `{VERSION_SLUG}` - URL-safe version (e.g., "1-2-3")
- `{VALHALLA_TOKEN}` - Your GitLab token

**Environment variables** (from CI/CD or system)
**Custom variables** (defined in your configuration)

### Can I override predefined variables?

No, predefined variables cannot be overridden. This is by design to prevent errors and ensure consistency.

### How do I use GitLab CI/CD variables in Valhalla?

Just reference them with curly braces:

```yaml
commit_before_release:
  before:
    - echo "Pipeline: {CI_PIPELINE_ID}"
    - echo "Commit: {CI_COMMIT_SHA}"
    - echo "Project: {CI_PROJECT_NAME}"
```

All GitLab CI/CD predefined variables are available.

### What happens if a variable is undefined?

If a variable is undefined, Valhalla will:
1. **Keep the placeholder** as-is (e.g., `{UNDEFINED_VAR}`)
2. **Continue execution** (may cause issues in commands)
3. **Log a warning** (if debug mode is enabled)

Always define required variables to avoid issues.

## CI/CD Integration

### Why do I need `dependencies: []` in GitLab CI?

The `dependencies: []` prevents GitLab from downloading build artifacts, which can cause issues when Valhalla commits files:

```yaml
valhalla_release:
  dependencies: []  # Prevents artifact conflicts during git operations
```

### How do I prevent infinite pipeline loops?

Valhalla automatically adds `[VALHALLA SKIP]` to commit messages to prevent re-triggering pipelines. Ensure your workflow rules exclude these commits:

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_TITLE !~ /.*VALHALLA SKIP.*/
```

### Can I use Valhalla with GitLab merge request pipelines?

Yes, but exclude merge request events in your job rules:

```yaml
valhalla_release:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - if: $CI_COMMIT_BRANCH =~ /^release-*/
```

### How do I handle different GitLab instances?

Valhalla automatically detects your GitLab instance from the git remote URL. It works with:
- GitLab.com
- Self-hosted GitLab instances
- GitLab Enterprise Edition

## Troubleshooting

### "No version to release found" error

**Causes and solutions:**

1. **Branch name doesn't match pattern:**
   ```bash
   # Wrong: feature/new-release
   # Right: release-1.2.3
   git checkout -b release-1.2.3
   ```

2. **Environment variable not set:**
   ```bash
   export VALHALLA_RELEASE_CMD=release-1.2.3
   ```

3. **No release configuration files:**
   ```bash
   # Check if valhalla.yml exists
   ls valhalla*.yml
   ```

### "Authentication failed" error

**Causes and solutions:**

1. **Token not set:**
   ```bash
   echo $VALHALLA_TOKEN  # Should output your token
   ```

2. **Token has insufficient permissions:**
   - Regenerate token with `api` and `write_repository` scopes

3. **Token expired:**
   - Check token expiration in GitLab settings
   - Generate a new token

### "Permission denied" during git operations

**Causes and solutions:**

1. **Repository is protected:**
   - Check branch protection rules
   - Ensure token has push permissions

2. **Git configuration missing:**
   ```yaml
   commit_before_release:
     username: "Release Bot"  # Required
     email: "bot@company.com"  # Required
   ```

### "Command failed" in before/after scripts

**Debugging steps:**

1. **Check command syntax:**
   ```yaml
   commit_before_release:
     before:
       - echo "Debug: Current directory is $(pwd)"
       - echo "Debug: Files present: $(ls -la)"
       - your-actual-command
   ```

2. **Verify environment:**
   ```yaml
   commit_before_release:
     before:
       - which docker  # Check if docker is available
       - node --version  # Check Node.js version
   ```

3. **Handle errors gracefully:**
   ```yaml
   commit_before_release:
     before:
       - your-command || echo "Command failed but continuing"
   ```

### "Configuration file inheritance failed"

**Causes and solutions:**

1. **URL not accessible:**
   ```yaml
   # Test URL accessibility
   extends:
     - https://raw.githubusercontent.com/company/configs/main/valhalla-base.yml
   ```

2. **Invalid YAML syntax:**
   - Validate your YAML with an online validator
   - Check for indentation issues

3. **Circular dependencies:**
   ```yaml
   # parent.yml extends child.yml and child.yml extends parent.yml
   # This is not supported!
   ```

## Advanced Usage

### Can I use Valhalla with monorepos?

Yes! You can create different configurations for different components:

```yaml
# valhalla-frontend.yml
variables:
  COMPONENT: "frontend"
  BUILD_PATH: "./frontend"

commit_before_release:
  before:
    - cd {BUILD_PATH} && npm run build
```

```yaml
# valhalla-backend.yml  
variables:
  COMPONENT: "backend"
  BUILD_PATH: "./backend"

commit_before_release:
  before:
    - cd {BUILD_PATH} && mvn clean package

# Use with: release-frontend-1.2.3, release-backend-2.1.0
```

### How do I integrate with external systems?

Use webhook calls and API integrations:

```yaml
commit_before_release:
  before:
    # Slack notification
    - curl -X POST "{SLACK_WEBHOOK}" -d '{"text":"Starting release {VERSION}"}'
    
    # Jira version creation
    - |
      curl -X POST "{JIRA_URL}/rest/api/3/version" \
      -H "Authorization: Bearer {JIRA_TOKEN}" \
      -d '{"name":"{VERSION}","projectId":"{JIRA_PROJECT_ID}"}'
    
    # Custom deployment system
    - curl -X POST "{DEPLOY_API}/prepare" -d '{"version":"{VERSION}"}'
```

### Can I use conditional logic in commands?

Yes! Use shell scripting:

```yaml
commit_before_release:
  before:
    - |
      if [ "{VERSION}" == *"RC"* ]; then
        echo "Release candidate - deploying to staging"
        kubectl apply -f staging-deployment.yml
      else
        echo "Production release - deploying to production"
        kubectl apply -f prod-deployment.yml
      fi
```

### How do I handle secrets in commands?

Use environment variables, not configuration files:

```yaml
# ❌ Never do this - hardcoded secrets
commit_before_release:
  before:
    - curl -H "Authorization: Bearer my-secret-token" api.example.com
```

```yaml
# ✅ Do this instead - use environment variables
commit_before_release:
  before:
    - curl -H "Authorization: Bearer {SECRET_TOKEN}" api.example.com
```

## Limitations and Known Issues

### What are Valhalla's current limitations?

1. **Platform support**: Only GitLab is currently supported
2. **Single inheritance**: Configuration can only extend from one URL
3. **List merging**: Lists are replaced, not merged during inheritance
4. **No rollback automation**: Manual rollback process required
5. **Sequential execution**: Commands run sequentially, not in parallel

### Known issues and workarounds

1. **Large file commits may timeout:**
   - Use `.gitignore` to exclude large generated files
   - Implement file cleanup in your scripts

2. **Docker layer caching issues:**
   - Use `dependencies: []` in GitLab CI
   - Clear Docker cache if needed: `docker system prune -f`

3. **Variable resolution in multi-line commands:**
   ```yaml
   # ❌ May not work as expected
   commit_before_release:
     before:
       - |
         echo "Version: {VERSION}"
         echo "Date: $(date)"
   ```
   
   ```yaml
   # ✅ Better approach
   commit_before_release:
     before:
       - echo "Version: {VERSION}"
       - echo "Date: $(date)"
   ```

### Future features and roadmap

Planned features include:
- GitHub support
- Parallel command execution
- Enhanced rollback capabilities
- Configuration validation
- Multi-inheritance support

### How can I contribute or report issues?

- 🐛 **Report bugs**: [GitHub Issues](https://github.com/logchange/valhalla/issues)
- 💡 **Feature requests**: [GitHub Discussions](https://github.com/logchange/valhalla/discussions)
- 🔧 **Contribute code**: [GitHub Pull Requests](https://github.com/logchange/valhalla/pulls)
- ⭐ **Support the project**: Give us a star on GitHub!

---

**Still have questions?** Check the [Usage Guide](usage.md) for more examples or [Reference](reference.md) for detailed configuration options.