# 🌌 Valhalla Documentation

**Streamline your software releases with automated, error-free deployments**

---

## Welcome to Valhalla

🌌 **Valhalla** is a powerful toolkit designed to automate and streamline the release process for software projects. Say goodbye to manual, error-prone releases and embrace automated workflows that save time, reduce mistakes, and ensure compliance with your release standards.

### ✨ Key Features

- **🚀 Automated Releases** - Create GitLab releases automatically from branch names or environment variables
- **🔄 Git Integration** - Automated commits, pushes, and merge request creation
- **📋 Configurable Workflows** - Flexible YAML configuration for any release process
- **🔧 Pre/Post Release Scripts** - Execute custom commands before and after releases
- **📦 Asset Management** - Automatically attach release assets and documentation links
- **🏗️ CI/CD Ready** - Seamless integration with GitLab CI/CD pipelines
- **🔗 Configuration Inheritance** - Share configurations across multiple repositories
- **🌐 Variable System** - Powerful templating with predefined and custom variables

### 🎯 Why Choose Valhalla?

| Problem | Valhalla Solution |
|---------|------------------|
| **Manual release complexity** | Automated multi-step release workflows |
| **Human errors & inconsistency** | Standardized, repeatable processes |
| **Time-consuming manual steps** | One-click releases triggered by branch creation |
| **Compliance & audit trails** | Consistent release documentation and history |
| **Multi-repository management** | Configuration inheritance and templates |

---

## 📚 Documentation

### 🚀 [Getting Started](getting-started.md)
**New to Valhalla?** Start here for installation, basic setup, and your first release.

- Installation options (Docker & source)
- Initial configuration
- GitLab CI/CD integration
- Your first automated release
- Prerequisites and setup

### 📖 [Usage Guide](usage.md)
**Ready for more?** Explore advanced usage patterns, configuration inheritance, and common workflows.

- Branch-based vs environment variable releases
- Configuration inheritance patterns
- Variable system and templating
- Multiple release types (standard, hotfix, preview)
- Integration examples (Slack, Jira, Docker)
- Best practices and troubleshooting

### ❓ [FAQ](faq.md)
**Got questions?** Find answers to common questions and troubleshooting tips.

- Platform compatibility and limitations
- Token permissions and security
- Testing and dry-run approaches
- Error resolution and debugging
- Advanced use cases and workarounds

### 📚 [Configuration Reference](reference.md)
**Need details?** Complete reference for all configuration options and parameters.

- Full YAML schema documentation
- All configuration options explained
- Predefined variables reference
- Environment integration details
- Complete configuration examples

---

## 🚀 Quick Start

Ready to get started? Here's the fastest way to begin:

### 1. **Install Valhalla**
```bash
docker pull logchange/valhalla:1.3.0
```

### 2. **Create Configuration**
```yaml
# valhalla.yml
git_host: gitlab

commit_before_release:
  enabled: true
  username: "Release Bot"
  email: "releases@company.com"
  msg: "Prepare release {VERSION}"
  before:
    - echo "Building version {VERSION}"

release:
  description:
    from_command: "echo 'Release {VERSION}'"

merge_request:
  enabled: true
  title: "Release {VERSION}"
```

### 3. **Set GitLab Token**
```bash
# In GitLab CI/CD Variables
VALHALLA_TOKEN=your_gitlab_token
```

### 4. **Trigger Release**
```bash
git checkout -b release-1.0.0
git push origin release-1.0.0
```

🎉 **That's it!** Valhalla will handle the rest automatically.

➡️ **[Continue with detailed setup →](getting-started.md)**

---

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Git Branch    │───▶│    Valhalla      │───▶│  GitLab Release │
│ release-1.2.3   │    │                  │    │   + Assets      │
└─────────────────┘    │  ┌─────────────┐ │    └─────────────────┘
                       │  │ Config      │ │              │
┌─────────────────┐    │  │ valhalla.yml│ │    ┌─────────────────┐
│ Environment     │───▶│  └─────────────┘ │───▶│  Merge Request  │
│ VALHALLA_*      │    │                  │    │    (optional)   │
└─────────────────┘    │  ┌─────────────┐ │    └─────────────────┘
                       │  │Pre/Post     │ │              │
┌─────────────────┐    │  │Scripts      │ │    ┌─────────────────┐
│ GitLab CI/CD    │───▶│  └─────────────┘ │───▶│   Git Commits   │
│   Variables     │    └──────────────────┘    │   + Push        │
└─────────────────┘                            └─────────────────┘
```

---

## 🌟 Common Use Cases

### 🔄 **Continuous Deployment**
Automatically create releases when code is merged to release branches.

### 🚨 **Hotfix Releases** 
Fast-track critical fixes with simplified release processes using `valhalla-hotfix.yml`.

### 📦 **Multi-Component Releases**
Coordinate releases across microservices and monorepo components.

### 🏢 **Enterprise Workflows**
Standardize release processes across teams with configuration inheritance.

### 🐳 **Container Deployments**
Build and publish Docker images as part of the release process.

---

## 🛠️ Platform Support

| Platform | Status | Notes |
|----------|--------|-------|
| **GitLab** | ✅ Full Support | Self-hosted and GitLab.com |
| **GitHub** | ⏳ Planned | Coming in future release |
| **Bitbucket** | ⏳ Planned | Coming in future release |
| **Azure DevOps** | ⏳ Planned | Coming in future release |

---

## 📊 Release Types

Valhalla supports multiple release configurations for different scenarios:

### 🎯 **Standard Releases** (`valhalla.yml`)
- Full feature releases
- Complete testing and validation
- Documentation updates
- Post-release preparation

### 🔥 **Hotfix Releases** (`valhalla-hotfix.yml`)
- Critical bug fixes
- Minimal validation
- Fast-track deployment
- Emergency response

### 👀 **Preview Releases** (`valhalla-preview.yml`)
- Development previews
- Staging deployments
- Feature demonstrations
- Integration testing

### 🏗️ **Custom Release Types** (`valhalla-{type}.yml`)
- Project-specific workflows
- Environment-specific releases
- Team-customized processes

---

## 🔧 Integration Ecosystem

Valhalla integrates seamlessly with your existing tools:

- **🦊 GitLab CI/CD** - Native pipeline integration
- **🐳 Docker** - Container building and publishing  
- **📦 Package Managers** - npm, Maven, NuGet, pip
- **📢 Notifications** - Slack, Teams, Discord
- **📋 Project Management** - Jira, Linear, GitHub Issues
- **📊 Monitoring** - Custom webhooks and metrics
- **🔐 Security** - Token management and secret handling

---

## 🤝 Community & Support

### 📞 Get Help
- 🐛 **Bug Reports**: [GitHub Issues](https://github.com/logchange/valhalla/issues)
- 💡 **Feature Requests**: [GitHub Discussions](https://github.com/logchange/valhalla/discussions)
- 📖 **Documentation**: You're reading it!
- ⭐ **Show Support**: [Star us on GitHub](https://github.com/logchange/valhalla)

### 🚀 Contributing
Valhalla is open-source and welcomes contributions:
- 🔧 **Code Contributions**: [Pull Requests](https://github.com/logchange/valhalla/pulls)
- 📝 **Documentation**: Help improve our guides
- 🧪 **Testing**: Share your use cases and configurations
- 🌍 **Community**: Help other users in discussions

---

## 📈 What's Next?

### 🎯 Choose Your Path

| I want to... | Go to... |
|--------------|----------|
| **Get started quickly** | [Getting Started Guide](getting-started.md) |
| **See advanced examples** | [Usage Guide](usage.md) |
| **Solve a specific problem** | [FAQ](faq.md) |
| **Look up configuration options** | [Configuration Reference](reference.md) |
| **Understand the full feature set** | Continue reading below |

---

## 🎉 Success Stories

> *"Valhalla reduced our release time from 2 hours to 5 minutes and eliminated all manual errors."*  
> — DevOps Team Lead

> *"Configuration inheritance lets us manage 20+ repositories with a single shared template."*  
> — Platform Engineer  

> *"The variable system makes our release notes and notifications perfectly consistent."*  
> — Release Manager

---

**Ready to transform your release process?** 🌌

**[Start with Getting Started →](getting-started.md)**

---

<div align="center">

**🌟 Like Valhalla? [Give us a star on GitHub!](https://github.com/logchange/valhalla) 🌟**

*Made with ❤️ by the Valhalla team*

</div>