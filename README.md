<div align="center">

<!-- Animated Typing Banner -->
<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=28&duration=3000&pause=1000&color=2E9EF7&center=true&vCenter=true&multiline=true&repeat=true&width=600&height=100&lines=Api+Design+Assistant;7+Agents+%7C+7+Skills;Claude+Code+Plugin" alt="Api Design Assistant" />

<br/>

<!-- Badge Row 1: Status Badges -->
[![Version](https://img.shields.io/badge/Version-2.0.0-blue?style=for-the-badge)](https://github.com/pluginagentmarketplace/custom-plugin-api-design/releases)
[![License](https://img.shields.io/badge/License-Custom-yellow?style=for-the-badge)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production-brightgreen?style=for-the-badge)](#)
[![SASMP](https://img.shields.io/badge/SASMP-v1.3.0-blueviolet?style=for-the-badge)](#)

<!-- Badge Row 2: Content Badges -->
[![Agents](https://img.shields.io/badge/Agents-7-orange?style=flat-square&logo=robot)](#-agents)
[![Skills](https://img.shields.io/badge/Skills-7-purple?style=flat-square&logo=lightning)](#-skills)
[![Commands](https://img.shields.io/badge/Commands-4-green?style=flat-square&logo=terminal)](#-commands)

<br/>

<!-- Quick CTA Row -->
[📦 **Install Now**](#-quick-start) · [🤖 **Explore Agents**](#-agents) · [📖 **Documentation**](#-documentation) · [⭐ **Star this repo**](https://github.com/pluginagentmarketplace/custom-plugin-api-design)

---

### What is this?

> **Api Design Assistant** is a Claude Code plugin with **7 agents** and **7 skills** for api design development.

</div>

---

## 📑 Table of Contents

<details>
<summary>Click to expand</summary>

- [Quick Start](#-quick-start)
- [Features](#-features)
- [Agents](#-agents)
- [Skills](#-skills)
- [Commands](#-commands)
- [Documentation](#-documentation)
- [Contributing](#-contributing)
- [License](#-license)

</details>

---

## 🚀 Quick Start

### Prerequisites

- Claude Code CLI v2.0.27+
- Active Claude subscription

### Installation (Choose One)

<details open>
<summary><strong>Option 1: From Marketplace (Recommended)</strong></summary>

```bash
# Step 1️⃣ Add the marketplace
/plugin marketplace add pluginagentmarketplace/custom-plugin-api-design

# Step 2️⃣ Install the plugin
/plugin install custom-plugin-api-design@pluginagentmarketplace-api-design

# Step 3️⃣ Restart Claude Code
# Close and reopen your terminal/IDE
```

</details>

<details>
<summary><strong>Option 2: Local Installation</strong></summary>

```bash
# Clone the repository
git clone https://github.com/pluginagentmarketplace/custom-plugin-api-design.git
cd custom-plugin-api-design

# Load locally
/plugin load .

# Restart Claude Code
```

</details>

### ✅ Verify Installation

After restart, you should see these agents:

```
custom-plugin-api-design:02-backend-patterns
custom-plugin-api-design:05-security-compliance
custom-plugin-api-design:04-devops-infrastructure
custom-plugin-api-design:06-frontend-integration
custom-plugin-api-design:03-database-performance
... and 2 more
```

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🤖 **7 Agents** | Specialized AI agents for api design tasks |
| 🛠️ **7 Skills** | Reusable capabilities with Golden Format |
| ⌨️ **4 Commands** | Quick slash commands |
| 🔄 **SASMP v1.3.0** | Full protocol compliance |

---

## 🤖 Agents

### 7 Specialized Agents

| # | Agent | Purpose |
|---|-------|---------|
| 1 | **02-backend-patterns** | Backend development expertise covering Node.js, Python, Go,  |
| 2 | **05-security-compliance** | Security architecture, authentication, authorization, encryp |
| 3 | **04-devops-infrastructure** | Infrastructure, deployment, and operations - Docker, Kuberne |
| 4 | **06-frontend-integration** | Frontend development and API consumption - React, TypeScript |
| 5 | **03-database-performance** | Database design, optimization, and performance tuning - SQL, |
| 6 | **07-scaling-patterns** | Enterprise patterns for scaling - microservices, async opera |
| 7 | **01-api-architect** | Expert in API architecture, contract design, versioning stra |

---

## 🛠️ Skills

### Available Skills

| Skill | Description | Invoke |
|-------|-------------|--------|
| `scaling-patterns` | Enterprise scaling patterns for microservices, event-driven  | `Skill("custom-plugin-api-design:scaling-patterns")` |
| `database-patterns` | Database design, optimization, and caching strategies for SQ | `Skill("custom-plugin-api-design:database-patterns")` |
| `backend-patterns` | Production-grade backend patterns for Node.js, Python, Go, a | `Skill("custom-plugin-api-design:backend-patterns")` |
| `api-architecture` | REST, GraphQL, and hybrid API architecture patterns for buil | `Skill("custom-plugin-api-design:api-architecture")` |
| `devops-patterns` | Infrastructure, deployment, and operations patterns for Dock | `Skill("custom-plugin-api-design:devops-patterns")` |
| `security-patterns` | Security architecture, authentication, authorization, and co | `Skill("custom-plugin-api-design:security-patterns")` |
| `frontend-patterns` | Frontend development and API integration patterns for React, | `Skill("custom-plugin-api-design:frontend-patterns")` |

---

## ⌨️ Commands

| Command | Description |
|---------|-------------|
| `/architect` | Design Your Plugin API System |
| `/audit` | Audit Your API Design |
| `/roadmap` | Get Implementation Roadmap |
| `/secure` | Security & Compliance Guide |

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute |
| [LICENSE](LICENSE) | License information |

---

## 📁 Project Structure

<details>
<summary>Click to expand</summary>

```
custom-plugin-api-design/
├── 📁 .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── 📁 agents/              # 7 agents
├── 📁 skills/              # 7 skills (Golden Format)
├── 📁 commands/            # 4 commands
├── 📁 hooks/
├── 📄 README.md
├── 📄 CHANGELOG.md
└── 📄 LICENSE
```

</details>

---

## 📅 Metadata

| Field | Value |
|-------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | 2025-12-29 |
| **Status** | Production Ready |
| **SASMP** | v1.3.0 |
| **Agents** | 7 |
| **Skills** | 7 |
| **Commands** | 4 |

---

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md).

1. Fork the repository
2. Create your feature branch
3. Follow the Golden Format for new skills
4. Submit a pull request

---

## ⚠️ Security

> **Important:** This repository contains third-party code and dependencies.
>
> - ✅ Always review code before using in production
> - ✅ Check dependencies for known vulnerabilities
> - ✅ Follow security best practices
> - ✅ Report security issues privately via [Issues](../../issues)

---

## 📝 License

Copyright © 2025 **Dr. Umit Kacar** & **Muhsin Elcicek**

Custom License - See [LICENSE](LICENSE) for details.

---

## 👥 Contributors

<table>
<tr>
<td align="center">
<strong>Dr. Umit Kacar</strong><br/>
Senior AI Researcher & Engineer
</td>
<td align="center">
<strong>Muhsin Elcicek</strong><br/>
Senior Software Architect
</td>
</tr>
</table>

---

<div align="center">

**Made with ❤️ for the Claude Code Community**

[![GitHub](https://img.shields.io/badge/GitHub-pluginagentmarketplace-black?style=for-the-badge&logo=github)](https://github.com/pluginagentmarketplace)

</div>
