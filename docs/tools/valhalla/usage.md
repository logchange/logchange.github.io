# 📖 Usage Guide

> GitHub repository: https://github.com/logchange/valhalla

This guide covers advanced usage scenarios, best practices, and common workflows for Valhalla.

## Table of Contents

- [Basic Usage Patterns](#basic-usage-patterns)
- [Configuration Inheritance](#configuration-inheritance)
- [Variable System](#variable-system)
- [Release Types](#release-types)
- [Common Workflows](#common-workflows)
- [Integration Examples](#integration-examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Basic Usage Patterns

### 1. Branch-Based Releases (Recommended)

The most common way to trigger releases is through branch naming:

```bash
# Standard release
git checkout -b release-2.1.0
git push origin release-2.1.0

# Release candidate
git checkout -b release-2.1.0-RC1
git push origin release-2.1.0-RC1

# Hotfix release (uses valhalla-hotfix.yml if present)
git checkout -b release-hotfix-2.0.1
git push origin release-hotfix-2.0.1
```

### 2. Environment Variable Releases

For more control or automation from external systems:

```bash
# In GitLab CI/CD variables or local environment
export VALHALLA_RELEASE_CMD=release-2.1.0

# Then trigger any pipeline
git commit --allow-empty -m "Trigger release"
git push origin main
```

### 3. Manual Execution

For testing or local development:

```bash
# Using Docker
export VALHALLA_TOKEN=your_token
export VALHALLA_RELEASE_CMD=release-1.0.0
docker run --rm -v $(pwd):/workspace -w /workspace \
  -e VALHALLA_TOKEN -e VALHALLA_RELEASE_CMD \
  logchange/valhalla:1.3.0

# Using Python directly
export VALHALLA_TOKEN=your_token
export VALHALLA_RELEASE_CMD=release-1.0.0
python -m valhalla
```

## Configuration Inheritance

Valhalla supports configuration inheritance to manage multiple repositories efficiently.

### Parent Configuration (valhalla-base.yml)

```yaml
# Shared configuration for all projects
variables:
  COMPANY_NAME: "Your Company"
  NOTIFICATION_SLACK: "#releases"

commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "releases@company.com"
  msg: "🚀 Preparing release {VERSION}"

release:
  description:
    from_command: "echo 'Release {VERSION} by {COMPANY_NAME}'"

merge_request:
  enabled: true
  title: "📦 Release {VERSION}"
  description: |
    ## Release {VERSION}
    
    This automated release was created by Valhalla.
    
    **Changes:**
    - See commit history for details
    
    **Notification:** {NOTIFICATION_SLACK}
```

### Child Configuration (valhalla.yml)

```yaml
# Project-specific configuration
extends:
  - https://raw.githubusercontent.com/company/configs/main/valhalla-base.yml

variables:
  PROJECT_TYPE: "microservice"
  
commit_before_release:
  before:
    - mvn versions:set -DnewVersion={VERSION}
    - mvn clean package -DskipTests
    - echo "Built {PROJECT_TYPE} version {VERSION}" > build-info.txt

release:
  description:
    from_command: "cat CHANGELOG.md | head -50"
  assets:
    links:
      - name: "JAR File"
        url: "https://nexus.company.com/repository/releases/com/company/app/{VERSION}/app-{VERSION}.jar"
        link_type: package
```

## Variable System

Valhalla provides a powerful variable substitution system with hierarchy support.

### Variable Hierarchy (Most to Least Important)

1. **Predefined Variables** (cannot be overridden)
2. **Custom Variables** (defined in `valhalla.yml`)
3. **Environment Variables** (from system/CI)

### Predefined Variables

```yaml
# Available in all string contexts
variables_example:
  version_info: "{VERSION}"           # e.g., "2.1.0"
  major_version: "{VERSION_MAJOR}"    # e.g., "2"
  minor_version: "{VERSION_MINOR}"    # e.g., "1"  
  patch_version: "{VERSION_PATCH}"    # e.g., "0"
  slug_version: "{VERSION_SLUG}"      # e.g., "2-1-0" (URL-safe)
  token: "{VALHALLA_TOKEN}"           # Your GitLab token
```

### Custom Variables

```yaml
variables:
  PROJECT_NAME: "My Amazing App"
  DOCKER_REGISTRY: "registry.company.com"
  MAINTAINER: "team-backend@company.com"
  BUILD_DATE: "{CI_COMMIT_TIMESTAMP}"  # GitLab CI variable

commit_before_release:
  before:
    - echo "Building {PROJECT_NAME} version {VERSION}"
    - docker build -t {DOCKER_REGISTRY}/{PROJECT_NAME}:{VERSION} .
    - docker push {DOCKER_REGISTRY}/{PROJECT_NAME}:{VERSION}

release:
  description:
    from_command: |
      echo "## {PROJECT_NAME} {VERSION}
      
      **Build Date:** {BUILD_DATE}
      **Maintainer:** {MAINTAINER}
      **Docker Image:** {DOCKER_REGISTRY}/{PROJECT_NAME}:{VERSION}"
```

### Environment Variables

Use any environment variable from your system or CI/CD:

```yaml
commit_before_release:
  before:
    # GitLab CI variables
    - echo "Pipeline: {CI_PIPELINE_ID}"
    - echo "Commit: {CI_COMMIT_SHA}"
    - echo "User: {GITLAB_USER_NAME}"
    
    # Custom environment variables
    - echo "Deployment target: {DEPLOY_ENVIRONMENT}"
    - curl -X POST "{SLACK_WEBHOOK_URL}" -d '{"text":"Releasing {VERSION}"}'
```

## Release Types

Valhalla supports multiple release configurations for different workflows.

### Standard Release (valhalla.yml)

```yaml
git_host: gitlab

commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "releases@company.com"
  msg: "Prepare release {VERSION}"
  before:
    - npm version {VERSION} --no-git-tag-version
    - npm run build
    - npm run test

release:
  description:
    from_command: "npm run changelog:generate"
  milestones:
    - "v{VERSION_MAJOR}.{VERSION_MINOR}"
    - "Sprint {CI_COMMIT_TIMESTAMP}"

commit_after_release:
  enabled: true
  username: "Release Bot"  
  email: "releases@company.com"
  msg: "Prepare next development iteration"
  before:
    - npm version patch --no-git-tag-version
    - git add package.json

merge_request:
  enabled: true
  title: "Release {VERSION}"
  reviewers:
    - "tech-lead"
    - "product-owner"
```

### Hotfix Release (valhalla-hotfix.yml)

```yaml
git_host: gitlab

# Simplified hotfix process - no development iteration prep
commit_before_release:
  enabled: true
  username: "Hotfix Bot"
  email: "hotfix@company.com"
  msg: "🔥 Hotfix {VERSION}"
  before:
    - npm version {VERSION} --no-git-tag-version
    - npm run build
    - npm run test:critical

release:
  description:
    from_command: "echo '🔥 Critical hotfix {VERSION}' && cat HOTFIX_NOTES.md"
  
# No commit_after_release for hotfixes

merge_request:
  enabled: true
  title: "🔥 HOTFIX {VERSION}"
  target_branch: "main"  # Merge directly to main
  description: "Critical hotfix - requires immediate review"
  reviewers:
    - "tech-lead"
    - "ops-team"
```

### Preview Release (valhalla-preview.yml)

```yaml
git_host: gitlab

commit_before_release:
  enabled: true
  username: "Preview Bot"
  email: "preview@company.com"
  msg: "Preview release {VERSION}"
  before:
    - docker build -t preview-app:{VERSION} .
    - docker push preview-registry/app:{VERSION}
    - kubectl apply -f preview-deployment.yml

release:
  description:
    from_command: "echo 'Preview deployment {VERSION}' && kubectl get pods -l app=preview-{VERSION_SLUG}"

# No merge request for preview releases
merge_request:
  enabled: false
```

## Common Workflows

### Maven Project Workflow

```yaml
git_host: gitlab

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

commit_before_release:
  enabled: true
  username: "Maven Release Bot"
  email: "maven-releases@company.com"
  msg: "Maven release {VERSION}"
  before:
    - mvn versions:set -DnewVersion={VERSION} -DgenerateBackupPoms=false
    - mvn clean compile test
    - mvn javadoc:javadoc
    - mvn package -DskipTests

release:
  description:
    from_command: "mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:evaluate -Dexpression=project.description -q -DforceStdout"
  assets:
    links:
      - name: "JAR File"
        url: "https://nexus.company.com/repository/releases/com/company/{CI_PROJECT_NAME}/{VERSION}/{CI_PROJECT_NAME}-{VERSION}.jar"
        link_type: package
      - name: "Javadoc"
        url: "https://nexus.company.com/repository/releases/com/company/{CI_PROJECT_NAME}/{VERSION}/{CI_PROJECT_NAME}-{VERSION}-javadoc.jar"
        link_type: other

commit_after_release:
  enabled: true
  username: "Maven Release Bot"
  email: "maven-releases@company.com"  
  msg: "Prepare for next development iteration"
  before:
    - mvn versions:set -DnewVersion={VERSION_MAJOR}.{VERSION_MINOR}.{VERSION_PATCH_NEXT}-SNAPSHOT -DgenerateBackupPoms=false
```

### Node.js Project Workflow

```yaml
git_host: gitlab

variables:
  NODE_VERSION: "18"
  NPM_REGISTRY: "https://npm.company.com"

commit_before_release:
  enabled: true
  username: "NPM Release Bot"
  email: "npm-releases@company.com"
  msg: "Release {VERSION}"
  before:
    - npm ci
    - npm version {VERSION} --no-git-tag-version
    - npm run build
    - npm run test
    - npm pack

release:
  description:
    from_command: "node -p 'require(\"./package.json\").description + \" - Version \" + require(\"./package.json\").version'"
  assets:
    links:
      - name: "NPM Package"  
        url: "{NPM_REGISTRY}/@company/{CI_PROJECT_NAME}/-/{CI_PROJECT_NAME}-{VERSION}.tgz"
        link_type: package

commit_after_release:
  enabled: true
  username: "NPM Release Bot"
  email: "npm-releases@company.com"
  msg: "Bump version for next development cycle"
  before:
    - npm version prerelease --preid=dev --no-git-tag-version
```

### Docker Image Workflow

```yaml
git_host: gitlab

variables:
  DOCKER_REGISTRY: "registry.company.com"
  IMAGE_NAME: "{CI_PROJECT_NAME}"

commit_before_release:
  enabled: true
  username: "Docker Release Bot"
  email: "docker-releases@company.com"
  msg: "Build Docker image {VERSION}"
  before:
    - echo "{VERSION}" > VERSION
    - docker build -t {DOCKER_REGISTRY}/{IMAGE_NAME}:{VERSION} .
    - docker build -t {DOCKER_REGISTRY}/{IMAGE_NAME}:latest .
    - docker push {DOCKER_REGISTRY}/{IMAGE_NAME}:{VERSION}
    - docker push {DOCKER_REGISTRY}/{IMAGE_NAME}:latest

release:
  description:
    from_command: "echo 'Docker image {DOCKER_REGISTRY}/{IMAGE_NAME}:{VERSION}' && docker inspect {DOCKER_REGISTRY}/{IMAGE_NAME}:{VERSION} --format='{{.Config.Labels}}'"
  assets:
    links:
      - name: "Docker Image"
        url: "https://registry.company.com/v2/{IMAGE_NAME}/tags/list"
        link_type: image
```

## Integration Examples

### Slack Notifications

```yaml
variables:
  SLACK_WEBHOOK: "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
  SLACK_CHANNEL: "#releases"

commit_before_release:
  before:
    - |
      curl -X POST "{SLACK_WEBHOOK}" \
      -H 'Content-type: application/json' \
      --data '{
        "channel": "{SLACK_CHANNEL}",
        "text": "🚀 Starting release {VERSION} for {CI_PROJECT_NAME}",
        "username": "Valhalla Bot"
      }'

commit_after_release:
  before:
    - |
      curl -X POST "{SLACK_WEBHOOK}" \
      -H 'Content-type: application/json' \
      --data '{
        "channel": "{SLACK_CHANNEL}",
        "text": "✅ Release {VERSION} completed for {CI_PROJECT_NAME}",
        "username": "Valhalla Bot"
      }'
```

### Jira Integration

```yaml
variables:
  JIRA_URL: "https://company.atlassian.net"
  JIRA_PROJECT: "PROJ"

commit_before_release:
  before:
    - |
      curl -X POST "{JIRA_URL}/rest/api/3/version" \
      -H "Authorization: Bearer {JIRA_TOKEN}" \
      -H "Content-Type: application/json" \
      --data '{
        "name": "{VERSION}",
        "projectId": "{JIRA_PROJECT_ID}",
        "released": true,
        "releaseDate": "{CI_COMMIT_TIMESTAMP}"
      }'
```

### Database Migration

```yaml
commit_before_release:
  before:
    - echo "Running database migrations for {VERSION}"
    - flyway -url=jdbc:postgresql://{DB_HOST}/prod -user={DB_USER} -password={DB_PASS} migrate
    - echo "Database migration completed"
```

## Best Practices

### 1. Configuration Management

- **Use inheritance** for shared configurations across repositories
- **Keep secrets in CI/CD variables**, not in configuration files
- **Version your configuration** alongside your code
- **Test configurations** in non-production environments first

### 2. Branch Management

- **Use consistent branch naming**: `release-*`, `release-hotfix-*`
- **Protect release branches** with merge request requirements
- **Clean up release branches** after successful releases
- **Use semantic versioning** for predictable releases

### 3. Release Notes

- **Automate changelog generation** using commit messages or external tools
- **Include migration notes** and breaking changes
- **Link to documentation** and deployment guides
- **Mention contributors** and acknowledgments

### 4. Asset Management

- **Upload relevant assets** (binaries, documentation, images)
- **Use descriptive names** and organize by type
- **Include checksums** for verification
- **Set proper permissions** for sensitive assets

### 5. Testing and Validation

- **Run comprehensive tests** before release creation
- **Validate configuration** in staging environments
- **Test rollback procedures** regularly
- **Monitor release success** with health checks

## Troubleshooting

### Common Issues

#### "No version to release found"

**Cause:** Branch name doesn't match expected pattern or `VALHALLA_RELEASE_CMD` not set.

**Solution:**
```bash
# Ensure branch name follows pattern
git branch -m my-branch release-1.2.3

# Or use environment variable
export VALHALLA_RELEASE_CMD=release-1.2.3
```

#### "Authentication failed"

**Cause:** `VALHALLA_TOKEN` missing or invalid.

**Solution:**
```bash
# Check token is set
echo $VALHALLA_TOKEN

# Verify token has correct permissions (api, write_repository)
# Regenerate token if necessary
```

#### "Configuration file not found"

**Cause:** No `valhalla.yml` file in repository root.

**Solution:**
```bash
# Create basic configuration
cat > valhalla.yml << EOF
git_host: gitlab
release:
  description:
    from_command: "echo 'Release {VERSION}'"
EOF
```

#### "Commit failed"

**Cause:** Git configuration missing or repository in dirty state.

**Solution:**
```bash
# Check git configuration
git config user.name
git config user.email

# Clean repository
git status
git stash  # if needed
```

### Debug Mode

Enable detailed logging by setting environment variables:

```bash
export VALHALLA_DEBUG=true
export VALHALLA_LOG_LEVEL=DEBUG

# Run with verbose output
docker run --rm -e VALHALLA_DEBUG -e VALHALLA_LOG_LEVEL \
  -v $(pwd):/workspace -w /workspace \
  logchange/valhalla:1.3.0
```

### Dry Run Mode

Test configuration without making actual changes:

```bash
# Add to valhalla.yml
variables:
  DRY_RUN: "true"

commit_before_release:
  before:
    - echo "DRY RUN: Would execute commands here"
    - if [ "{DRY_RUN}" = "true" ]; then echo "Skipping actual changes"; exit 0; fi
```

---

For more advanced scenarios and troubleshooting, check the [FAQ](faq.md) or visit the [GitHub Issues](https://github.com/logchange/valhalla/issues) page.