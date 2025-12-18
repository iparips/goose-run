# goose-run

Note: This project is a work in progress.

A Docker sandbox for running [Goose](https://github.com/square/goose) AI coding assistant with filesystem isolation and transparent CLI integration.

## Overview

goose-run provides a containerised environment for Goose that limits its access to only your current project directory, preventing unauthorised access to other parts of your filesystem. It offers a seamless command-line experience where `goose-run` behaves exactly like the native `goose` command.

## Key Features

- Filesystem Isolation: Container only accesses the directory where you invoke goose-run
- Transparent Wrapper: Use `goose-run` exactly like `goose` - all commands pass through seamlessly
- Persistent Configuration: Per-project `.goose` configuration stored in your workspace
- Fast Reconnection: Container stays running after session exit for quick reconnection
- Global Installation: Install once, use from any project directory
- ARM64 Optimised: Built for Apple Silicon with Debian Bookworm Slim base

## Security Model

- Dynamic mounting: Only `$(PWD)` is mounted as `/workspace` in the container
- No host filesystem access outside the mounted directory
- API keys passed via environment variables, never baked into images
- Configurable resource limits (16GB memory limit, 4GB reservation by default)

## Quick Start

### Prerequisites

- Docker
- docker-compose
- OpenRouter API key (set as `OPENROUTER_API_KEY` environment variable)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/goose-run.git
cd goose-run

# Run setup script to install goose-run globally
./setup.sh

# Set your API key (add to ~/.zshrc or ~/.bashrc)
export OPENROUTER_API_KEY="your-api-key-here"
```

### Usage

Navigate to any project and run Goose:

```bash
cd /path/to/your-project
goose-run session        # Start interactive session
goose-run --version      # Check Goose version
goose-run configure      # Configure Goose settings
goose-run --help         # Show help
```

### Container Management

```bash
goose-run stop           # Stop the container
goose-run restart        # Restart the container
goose-run status         # Check container status
```

## How It Works

1. You run `goose-run` from any project directory
2. The current directory is mounted as `/workspace` in the container
3. Container starts automatically (or attaches if already running)
4. All commands are passed to Goose inside the container
5. Goose configuration is stored in `/workspace/.goose` (persists in your project)
6. Exiting a session leaves the container running for quick reconnection

## Project Structure

```
goose-run/
├── spec/
│   └── requirements/     # Detailed requirements documentation
├── Dockerfile            # Container image definition
├── docker-compose.yml    # Container orchestration
├── setup.sh             # Installation script
└── bin/
    └── goose-run        # CLI wrapper script
```

## Configuration

The container creates these files in your workspace on first run:
- `.goose/` - Goose configuration directory
- `.goosehints` - Project context for Goose (optional)

Add these to your project's `.gitignore` if you don't want to commit them.

## Requirements

For detailed requirements, see [spec/requirements/INDEX.md](spec/requirements/INDEX.md).

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please read the requirements documentation before submitting PRs.
