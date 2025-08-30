# 📚 Configuration Reference

This reference provides complete documentation for all Valhalla configuration options, parameters, and features.

## Table of Contents

- [Configuration File Structure](#configuration-file-structure)
- [Root Level Options](#root-level-options)
- [Inheritance](#inheritance)
- [Variables](#variables)
- [Commit Configuration](#commit-configuration)
- [Release Configuration](#release-configuration)
- [Merge Request Configuration](#merge-request-configuration)
- [Predefined Variables](#predefined-variables)
- [Environment Integration](#environment-integration)

## Configuration File Structure

Valhalla uses YAML configuration files named according to the release type:

- `valhalla.yml` - Default configuration (triggered by `release-*` branches)
- `valhalla-{type}.yml` - Specific release types (e.g., `valhalla-hotfix.yml` for `release-hotfix-*`)

### Basic Structure

```yaml
# Optional: Inherit from parent configurations
extends:
  - https://example.com/parent-config.yml

# Optional: Custom variables for string interpolation
variables:
  KEY: "value"

# Required: Git hosting platform
git_host: gitlab

# Optional: Pre-release commit configuration
commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "bot@example.com"
  msg: "Prepare release {VERSION}"
  before:
    - echo "Pre-release commands"

# Required: Release configuration
release:
  description:
    from_command: "echo 'Release {VERSION}'"
  milestones:
    - "Milestone {VERSION}"
  assets:
    links:
      - name: "Asset Name"
        url: "https://example.com/asset"
        link_type: "package"

# Optional: Post-release commit configuration
commit_after_release:
  enabled: true
  username: "Release Bot"
  email: "bot@example.com" 
  msg: "Prepare next iteration"
  before:
    - echo "Post-release commands"

# Optional: Merge request configuration
merge_request:
  enabled: true
  target_branch: "main"
  title: "Release {VERSION}"
  description: "Automated release"
  reviewers:
    - "reviewer1"
    - "reviewer2"
```

## Root Level Options

### `git_host`

**Type:** `string`  
**Required:** Yes  
**Default:** None  
**Valid values:** `gitlab`

Specifies the Git hosting platform for API interactions.

```yaml
git_host: gitlab
```

> **Note:** Currently only GitLab is supported. GitHub, Bitbucket, and other platforms are planned for future releases.

## Inheritance

### `extends`

**Type:** `array[string]`  
**Required:** No  
**Default:** `[]`

Allows configuration inheritance from parent files via URLs.

```yaml
extends:
  - https://raw.githubusercontent.com/company/configs/main/base-config.yml
  - https://internal.company.com/valhalla/shared.yml
```

**Inheritance Rules:**
- Child configurations override parent values
- Arrays are completely replaced (not merged)  
- Only one level of inheritance supported
- Parent files cannot contain `extends` themselves

**Example:**

*Parent config:*
```yaml
variables:
  COMPANY: "Acme Corp"
  ENVIRONMENT: "production"

commit_before_release:
  msg: "Release by {COMPANY}"
```

*Child config:*
```yaml
extends:
  - https://example.com/parent.yml

variables:
  ENVIRONMENT: "staging"  # Overrides parent

# Inherits COMPANY and commit_before_release.msg from parent
```

## Variables

### `variables`

**Type:** `object`  
**Required:** No  
**Default:** `{}`

Defines custom variables for string interpolation throughout the configuration.

```yaml
variables:
  PROJECT_NAME: "My Application"
  DOCKER_REGISTRY: "registry.company.com"
  MAINTAINER_EMAIL: "team@company.com"
  BUILD_ARGS: "--optimization=true"
```

**Usage in strings:**
```yaml
commit_before_release:
  before:
    - echo "Building {PROJECT_NAME}"
    - docker build -t {DOCKER_REGISTRY}/{PROJECT_NAME}:{VERSION} {BUILD_ARGS} .
    - echo "Maintainer: {MAINTAINER_EMAIL}"
```

**Variable Resolution Order:**
1. **Predefined variables** (highest priority - cannot be overridden)
2. **Custom variables** (defined in `variables` section)
3. **Environment variables** (lowest priority)

## Commit Configuration

Both `commit_before_release` and `commit_after_release` share the same configuration structure.

### Commit Configuration Object

**Type:** `object`  
**Required:** No

#### `enabled`

**Type:** `boolean`  
**Required:** No  
**Default:** `false`

Controls whether the commit phase is executed.

```yaml
commit_before_release:
  enabled: true  # Enable this commit phase
```

#### `username`

**Type:** `string`  
**Required:** Yes (when `enabled: true`)  
**Default:** None

Git username for commits. Supports variable interpolation.

```yaml
commit_before_release:
  username: "Release Bot {VERSION}"
```

```yaml
commit_before_release:
  username: "{CI_PROJECT_NAME} Releaser"
```

#### `email`

**Type:** `string`  
**Required:** Yes (when `enabled: true`)  
**Default:** None

Git email for commits. Supports variable interpolation.

```yaml
commit_before_release:
  email: "releases@company.com"
```

```yaml
commit_before_release:
  email: "bot+{CI_PROJECT_NAME}@company.com"
```

#### `msg`

**Type:** `string`  
**Required:** Yes (when `enabled: true`)  
**Default:** None

Git commit message. Supports variable interpolation. Valhalla automatically appends `[VALHALLA SKIP]` to prevent pipeline loops.

```yaml
commit_before_release:
  msg: "Prepare release {VERSION}"
```

```yaml
commit_before_release:
  msg: "🚀 Release {VERSION} for {CI_PROJECT_NAME}"
```

#### `before`

**Type:** `array[string]`  
**Required:** No  
**Default:** `[]`

List of shell commands to execute before committing. Each command supports variable interpolation.

```yaml
commit_before_release:
  before:
    - echo "Starting release process for {VERSION}"
    - npm version {VERSION} --no-git-tag-version
    - npm run build
    - npm test
    - echo "Build completed at $(date)"
```

**Command Execution:**
- Commands run sequentially in order
- Failure of any command stops the entire process
- Commands run in the repository root directory
- Full shell functionality available (pipes, redirects, etc.)

### `commit_before_release`

Executed after version detection but before release creation.

**Common use cases:**
- Update version files
- Build artifacts
- Run tests
- Generate documentation
- Create release notes

### `commit_after_release`

Executed after successful release creation.

**Common use cases:**
- Prepare next development iteration
- Clean up build artifacts  
- Update development versions
- Create post-release documentation

## Release Configuration

### `release`

**Type:** `object`  
**Required:** Yes

Core release configuration defining how the GitLab release is created.

#### `description`

**Type:** `object`  
**Required:** Yes

Defines the release description/notes.

##### `from_command`

**Type:** `string`  
**Required:** Yes  

Shell command whose output becomes the release description. Supports variable interpolation.

```yaml
release:
  description:
    from_command: "cat CHANGELOG.md"
```

```yaml
release:
  description:
    from_command: "echo 'Release {VERSION}' && cat release-notes/{VERSION}.md"
```

```yaml
release:
  description:
    from_command: "python scripts/generate_notes.py {VERSION}"
```

#### `milestones`

**Type:** `array[string]`  
**Required:** No  
**Default:** `[]`

List of GitLab milestones to associate with the release. Supports variable interpolation.

```yaml
release:
  milestones:
    - "v{VERSION_MAJOR}.{VERSION_MINOR}"
    - "Release {VERSION}"
    - "Sprint-{CI_COMMIT_TIMESTAMP}"
```

**Notes:**
- Milestones must exist in the GitLab project
- Non-existent milestones are logged as warnings but don't fail the release
- Use milestone titles, not IDs

#### `assets`

**Type:** `object`  
**Required:** No  
**Default:** `{}`

Defines release assets and links.

##### `links`

**Type:** `array[object]`  
**Required:** No  
**Default:** `[]`

Array of link objects for release assets.

**Link Object:**

```yaml
release:
  assets:
    links:
      - name: "JAR File"                    # Link display name
        url: "https://repo.com/app.jar"     # Link URL  
        link_type: "package"                # Link type
        filepath: "/app-{VERSION}.jar"      # Optional: file path
      - name: "Documentation"
        url: "https://docs.company.com/{VERSION}"
        link_type: "other"
```

**Link Properties:**

- **`name`** (string, required): Display name for the link
- **`url`** (string, required): Target URL  
- **`link_type`** (string, required): Type of link
- **`filepath`** (string, optional): File path for downloads

**Valid `link_type` values:**
- `package` - Software packages, binaries
- `image` - Docker images, screenshots  
- `runbook` - Documentation, runbooks
- `other` - Other types of links

## Merge Request Configuration

### `merge_request`

**Type:** `object`  
**Required:** No

Configuration for automatic merge request creation after release.

#### `enabled`

**Type:** `boolean`  
**Required:** No  
**Default:** `false`

Controls whether merge request is created.

```yaml
merge_request:
  enabled: true
```

#### `target_branch`

**Type:** `string`  
**Required:** No  
**Default:** Default repository branch

Target branch for the merge request. Supports variable interpolation and regex patterns.

```yaml
merge_request:
  target_branch: "main"
```

```yaml
merge_request:
  target_branch: "develop"
```

```yaml
merge_request:
  target_branch: "hotfix-{VERSION}"
```

```yaml
merge_request:
  target_branch: "release-branch-.*"  # Regex pattern
```

#### `title`

**Type:** `string`  
**Required:** Yes (when `enabled: true`)  
**Default:** None

Merge request title. Supports variable interpolation.

```yaml
merge_request:
  title: "Release {VERSION}"
```

```yaml
merge_request:
  title: "🚀 {CI_PROJECT_NAME} {VERSION}"
```

```yaml
merge_request:
  title: "Merge release-{VERSION} to main"
```

#### `description`

**Type:** `string`  
**Required:** No  
**Default:** None

Merge request description. Supports variable interpolation and multi-line strings.

```yaml
merge_request:
  description: "Automated release of version {VERSION}"
```

```yaml
merge_request:
  description: |
    ## Release {VERSION}
    
    This release includes:
    - New features
    - Bug fixes
    - Performance improvements
    
    **Build:** {CI_PIPELINE_ID}
    **Commit:** {CI_COMMIT_SHA}
```

#### `reviewers`

**Type:** `array[string]`  
**Required:** No  
**Default:** `[]`

List of GitLab usernames to assign as reviewers.

```yaml
merge_request:
  reviewers:
    - "tech-lead"
    - "product-owner"
    - "senior-developer"
```

**Notes:**
- Uses GitLab usernames, not display names
- Invalid usernames are logged as warnings but don't fail the process
- Reviewers must have access to the repository

## Predefined Variables

Valhalla provides several predefined variables that are automatically available in all string contexts.

### Version Variables

These are extracted from the branch name or `VALHALLA_RELEASE_CMD` environment variable.

| Variable | Description | Example (for `release-2.1.3-RC1`) |
|----------|-------------|-----------------------------------|
| `{VERSION}` | Complete version string | `2.1.3-RC1` |
| `{VERSION_MAJOR}` | Major version number | `2` |
| `{VERSION_MINOR}` | Minor version number | `1` |
| `{VERSION_PATCH}` | Patch version number | `3` |
| `{VERSION_SLUG}` | URL-safe version | `2-1-3-rc1` |

**Version Slug Rules:**
- Converts to lowercase
- Replaces non-alphanumeric characters with hyphens
- Removes leading/trailing hyphens
- Useful for URLs, hostnames, file paths

### System Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{VALHALLA_TOKEN}` | GitLab access token | `glpat-xxxx...` |

**Security Note:** The token variable is automatically masked in logs and output for security.

### Branch-Based Configuration Selection

Valhalla automatically selects configuration files based on branch patterns:

| Branch Pattern | Configuration File | Example |
|----------------|-------------------|---------|
| `release-*` | `valhalla.yml` | `release-1.2.3` |
| `release-hotfix-*` | `valhalla-hotfix.yml` | `release-hotfix-1.2.4` |
| `release-preview-*` | `valhalla-preview.yml` | `release-preview-2.0.0` |
| `release-{type}-*` | `valhalla-{type}.yml` | `release-beta-1.0.0` → `valhalla-beta.yml` |

## Environment Integration

### GitLab CI/CD Variables

All GitLab CI/CD predefined variables are available in Valhalla configurations:

**Common GitLab Variables:**

| Variable | Description | Example |
|----------|-------------|---------|
| `{CI_PROJECT_NAME}` | Project name | `my-application` |
| `{CI_PROJECT_ID}` | Project ID | `123` |
| `{CI_COMMIT_SHA}` | Commit hash | `a1b2c3d4...` |
| `{CI_COMMIT_TIMESTAMP}` | Commit timestamp | `2024-01-15T10:30:00Z` |
| `{CI_PIPELINE_ID}` | Pipeline ID | `456789` |
| `{CI_JOB_ID}` | Job ID | `987654` |
| `{GITLAB_USER_NAME}` | User triggering job | `john.doe` |

**Usage Example:**
```yaml
commit_before_release:
  msg: "Release {VERSION} from pipeline {CI_PIPELINE_ID}"
  before:
    - echo "Released by {GITLAB_USER_NAME} at {CI_COMMIT_TIMESTAMP}"

release:
  description:
    from_command: "echo 'Project: {CI_PROJECT_NAME}' && echo 'Commit: {CI_COMMIT_SHA}'"
```

### Environment Variable Access

Access any environment variable using the `{VARIABLE_NAME}` syntax:

```yaml
variables:
  DEPLOYMENT_ENV: "{DEPLOY_TARGET}"  # Uses DEPLOY_TARGET env var
  
commit_before_release:
  before:
    - echo "Deploying to {DEPLOYMENT_ENV}"
    - curl -X POST "{WEBHOOK_URL}" -d '{"version":"{VERSION}"}'
```

### Required Environment Variables

- `VALHALLA_TOKEN`: GitLab access token (required for all operations)
- `VALHALLA_RELEASE_CMD`: Optional alternative to branch-based triggering

**Setting in GitLab CI/CD:**
1. Go to Project → Settings → CI/CD → Variables
2. Add `VALHALLA_TOKEN` with your GitLab token
3. Mark as "Protected" and "Masked" for security

## Configuration Validation

### Required Fields Validation

Valhalla validates configuration at runtime and will fail with descriptive errors if:

- `git_host` is missing or invalid
- `release.description.from_command` is missing
- Commit configuration is enabled but missing required fields (`username`, `email`, `msg`)
- Merge request is enabled but missing required fields (`title`)

### Best Practices

1. **Test configurations** in non-production environments first
2. **Use inheritance** for shared settings across repositories  
3. **Keep secrets in environment variables**, not configuration files
4. **Validate YAML syntax** before committing changes
5. **Use descriptive variable names** for maintainability
6. **Document custom variables** in comments

### Example: Complete Configuration

```yaml
# Complete Valhalla configuration example
extends:
  - https://raw.githubusercontent.com/company/configs/main/base-valhalla.yml

variables:
  PROJECT_TYPE: "microservice"
  DOCKER_REGISTRY: "registry.company.com" 
  MAVEN_OPTS: "-Xmx1024m"
  NOTIFICATION_CHANNEL: "#releases"

git_host: gitlab

commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "releases@company.com"
  msg: "🚀 Prepare release {VERSION} for {PROJECT_TYPE}"
  before:
    - echo "Starting release {VERSION} at $(date)"
    - mvn versions:set -DnewVersion={VERSION} -DgenerateBackupPoms=false
    - mvn clean test package {MAVEN_OPTS}
    - docker build -t {DOCKER_REGISTRY}/{CI_PROJECT_NAME}:{VERSION} .
    - docker push {DOCKER_REGISTRY}/{CI_PROJECT_NAME}:{VERSION}
    - echo "Build completed successfully"

release:
  description:
    from_command: |
      echo "## Release {VERSION}" && \
      echo "" && \
      echo "**Type:** {PROJECT_TYPE}" && \
      echo "**Build:** {CI_PIPELINE_ID}" && \
      echo "**Docker Image:** {DOCKER_REGISTRY}/{CI_PROJECT_NAME}:{VERSION}" && \
      echo "" && \
      cat CHANGELOG.md | head -20
  milestones:
    - "v{VERSION_MAJOR}.{VERSION_MINOR}"
    - "Release {VERSION}"
  assets:
    links:
      - name: "JAR Artifact"
        url: "https://nexus.company.com/repository/releases/com/company/{CI_PROJECT_NAME}/{VERSION}/{CI_PROJECT_NAME}-{VERSION}.jar"
        link_type: package
      - name: "Docker Image"
        url: "https://{DOCKER_REGISTRY}/v2/{CI_PROJECT_NAME}/tags/list"
        link_type: image
      - name: "Documentation"
        url: "https://docs.company.com/{CI_PROJECT_NAME}/{VERSION}/"
        link_type: other

commit_after_release:
  enabled: true
  username: "Release Bot"
  email: "releases@company.com"
  msg: "Prepare for next development iteration after {VERSION}"
  before:
    - mvn versions:set -DnewVersion={VERSION_MAJOR}.{VERSION_MINOR}.{VERSION_PATCH_NEXT}-SNAPSHOT -DgenerateBackupPoms=false
    - echo "Next development version set"

merge_request:
  enabled: true
  target_branch: "develop"
  title: "🔄 Merge release {VERSION} back to develop"
  description: |
    ## Post-Release Merge
    
    This MR merges changes from release {VERSION} back to the develop branch.
    
    **Release Notes:** See GitLab release for details
    **Pipeline:** {CI_PIPELINE_ID}
    **Notification:** {NOTIFICATION_CHANNEL}
  reviewers:
    - "tech-lead"
    - "senior-developer"
```

---

For more examples and use cases, see the [Usage Guide](usage.md) and [FAQ](faq.md).