# 🚀 Getting Started with Valhalla

Welcome to Valhalla! This guide will help you get up and running with automated software releases in no time.

## What is Valhalla?

🌌 Valhalla is a toolkit designed to streamline the release of new versions of software. It automates the complex, error-prone process of creating releases by:

- ✅ Eliminating manual release workflows
- ⚡ Reducing time spent on repetitive tasks
- 🛡️ Ensuring compliance with release standards
- 🔄 Automating git operations, release creation, and merge requests

## Prerequisites

Before getting started, ensure you have:

- **Git repository** hosted on GitLab (currently the only supported platform)
- **GitLab access token** with appropriate permissions
- **Docker** installed (recommended) or Python 3.x environment
- **Basic familiarity** with YAML configuration files

## Quick Installation

### Option 1: Using Docker (Recommended)

Valhalla is distributed as a Docker image for easy integration with CI/CD pipelines:

```bash
docker pull logchange/valhalla:1.3.0
```

### Option 2: From Source

If you prefer to run Valhalla locally:

```bash
git clone https://github.com/logchange/valhalla.git
cd valhalla
pip install -r requirements.txt
```

## Initial Setup

### 1. Create Your Configuration File

Create a `valhalla.yml` file in your project root:

```yaml
# This file is used by valhalla tool to create release 🌌
# Visit https://github.com/logchange/valhalla and leave a star 🌟

git_host: gitlab  # Currently only GitLab is supported

# Define actions to execute before creating the release
commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "releases@yourcompany.com"
  msg: "Preparing release {VERSION}"
  before:
    - echo "Building changelog for version {VERSION}"
    - mkdir -p releases/v{VERSION}
    - echo "Release {VERSION}" > releases/v{VERSION}/notes.md

# Configure the release itself
release:
  description:
    from_command: "cat releases/v{VERSION}/notes.md"

# Optional: Create merge request after release
merge_request:
  enabled: true
  title: "Release {VERSION} completed"
  description: "Automated release of version {VERSION}"
```

### 2. Set Up Access Token

Create a GitLab access token with the following scopes:
- `api` - Full API access
- `write_repository` - Write access to repository

**Add the token to your CI/CD environment:**

```bash
# In GitLab CI/CD variables
VALHALLA_TOKEN=your_gitlab_token_here
```

### 3. Configure GitLab CI/CD

Add this job to your `.gitlab-ci.yml`:

```yaml
# Add release stage
stages:
  - build
  - test
  - release  # Add this stage

# Configure workflow rules
workflow:
  rules:
    - if: $CI_COMMIT_BRANCH =~ /^release-*/ && $CI_COMMIT_TITLE !~ /.*VALHALLA SKIP.*/
    - if: $VALHALLA_RELEASE_CMD =~ /^release-*/ && $CI_COMMIT_TITLE !~ /.*VALHALLA SKIP.*/
    # ... your other workflow rules

# Valhalla release job
valhalla_release:
  stage: release
  image: logchange/valhalla:1.3.0
  dependencies: []  # Prevent artifact conflicts
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - if: $CI_PIPELINE_SOURCE == "schedule"
      when: never
    - if: $CI_COMMIT_BRANCH =~ /^release-*/ && $CI_COMMIT_TITLE !~ /.*VALHALLA SKIP.*/
    - if: $VALHALLA_RELEASE_CMD =~ /^release-*/ && $CI_COMMIT_TITLE !~ /.*VALHALLA SKIP.*/
```

### 4. Update .gitignore

Add these rules to prevent committing unwanted files:

```gitignore
### Valhalla ###
.m2/
```

## Your First Release

### Method 1: Release Branch (Recommended)

1. **Create a release branch:**
   ```bash
   git checkout -b release-1.0.0
   git push origin release-1.0.0
   ```

2. **Valhalla automatically detects the branch** and starts the release process! 🚀

### Method 2: Environment Variable

1. **Set the release command:**
   ```bash
   export VALHALLA_RELEASE_CMD=release-1.0.0
   ```

2. **Push any commit** to trigger the release process.

## What Happens During a Release?

When Valhalla runs, it follows this workflow:

1. 🔍 **Detect version** from branch name or environment variable
2. 🔑 **Authenticate** using your GitLab token
3. ⚙️ **Load configuration** from `valhalla.yml`
4. 📝 **Execute pre-release scripts** (if configured)
5. 🏷️ **Create GitLab release** with description and assets
6. 📝 **Execute post-release scripts** (if configured)
7. 🔄 **Create merge request** (if enabled)

## Version Format

Valhalla extracts version numbers from branch names:

- `release-1.2.3` → Version: `1.2.3`
- `release-2.0.0-RC1` → Version: `2.0.0-RC1`
- `release-hotfix-1.2.4` → Version: `1.2.4` (uses hotfix config)

## Next Steps

Now that you have Valhalla up and running:

1. 📖 Read the [Usage Guide](usage.md) for advanced configuration options
2. ❓ Check the [FAQ](faq.md) for common questions and troubleshooting
3. 📚 Explore the [Reference](reference.md) for complete configuration details

## Need Help?

- 🐛 **Issues:** [GitHub Issues](https://github.com/logchange/valhalla/issues)
- 📖 **Documentation:** Check our other guides in this repository
- ⭐ **Like Valhalla?** Give us a star on [GitHub](https://github.com/logchange/valhalla)!

---

**Ready to automate your releases?** 🌌 Let Valhalla handle the complexity while you focus on building great software!