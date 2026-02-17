<div align="center">

# Feline

**Agentic AI Coding Assistant**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![VS Code](https://img.shields.io/badge/VS%20Code-Extension-green.svg)](https://code.visualstudio.com/)

</div>

Feline is an open-source, agentic AI coding assistant with intelligent code generation, multi-model LLM support, and autonomous tool execution. Features real-time streaming, context compression, task planning, session backtracking, and self-hosted or cloud deployment options.

## ✨ Features

- **Multi-LLM Support** - Works with multiple language model providers with automatic retry and fallback
- **Autonomous Tool Execution** - File operations, terminal commands, Git integration, and code search
- **Intelligent Context Management** - Automatic context compression to maximize conversation history
- **Plan Mode** - Structured task planning with user confirmation before execution
- **Session Management** - Conversation backtracking, forking, and history persistence
- **Real-time Streaming** - Live response streaming with thinking process visibility
- **Mention System** - Reference files with `@filename` and symbols with `#symbol`
- **Self-hosted or Cloud** - Run locally with your own API keys or use cloud services

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    VS Code Extension                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │  Extension  │  │  Chat UI    │  │  Local Backend  │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    Cloud Services                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  API Server  │  │  Database    │  │  Monitoring  │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## 📦 Project Structure

```
feline/
├── feline-plugin/     # VS Code extension (core product)
│   ├── packages/
│   │   ├── feline/    # Extension host
│   │   ├── iris/      # Chat UI (React)
│   │   └── whisker/   # Local backend (Python)
├── feline-server/     # Cloud API server
├── feline-monitor/    # Monitoring dashboard
└── feline-deploy/     # Docker deployment
```

## 🚀 Quick Start

### Prerequisites

- Node.js >= 18
- Python >= 3.10

### Installation

```bash
# Clone the repository
git clone https://github.com/phanhom/feline.git
cd feline

# Install dependencies
npm run install:all
```

### Development

```bash
# Start extension development
npm run dev:feline

# Start UI development
npm run dev:iris

# Start backend development
npm run dev:whisker
```

### Build

```bash
# Build all packages
npm run build

# Package VS Code extension
npm run package:vsix
```

## 🐳 Docker Deployment

```bash
cd feline-deploy

# Configure environment
cp .env.example .env
vim .env

# Start all services
docker-compose up -d
```

## 📖 Documentation

- [Plugin README](feline-plugin/README.md) - Extension usage and features
- [Server README](feline-server/README.md) - API documentation
- [Deploy README](feline-deploy/README.md) - Deployment guide
- [Developer Guide](developer.md) - Project evaluation and roadmap

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
Made with ❤️ by the Feline Team
</div>
