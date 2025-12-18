# Goose Docker Sandbox - Requirements

## REQ-001: Container Base System

### Motivation
Provide a lightweight, stable, and ARM64-optimised base system for running Goose in an isolated Docker environment. The base must be minimal to reduce attack surface and image size whilst supporting all necessary development tools.

### In Scope
- Use Debian Bookworm Slim as base image
- Install essential development tools (git, curl, build-essential)
- Configure for ARM64 architecture (Apple Silicon)
- Set up non-interactive installation mode

### Acceptance Criteria

```gherkin
Scenario: Container builds successfully
  Given the Dockerfile uses debian:bookworm-slim
  When I run docker build
  Then the build completes without errors
  And the final image size is under 200MB

Scenario: Essential tools are available
  Given the container is running
  When I execute git --version
  Then I see git version information
  When I execute curl --version
  Then I see curl version information

Scenario: ARM64 architecture is supported
  Given I am building on Apple Silicon
  When I inspect the built image
  Then the architecture is linux/arm64
```

---

## REQ-002: Goose Installation

### Motivation
Install the latest stable version of Goose using the official installation method. Avoid Python dependencies since Goose is a Rust binary, reducing container size and complexity.

### In Scope
- Download and install Goose using official GitHub installation script
- Configure Goose to skip interactive setup during build
- Add Goose binary to system PATH
- Enable version updates via `goose update` command

### Acceptance Criteria

```gherkin
Scenario: Goose installs successfully
  Given the base system is set up
  When the official install script runs with CONFIGURE=false
  Then Goose binary is installed to /root/.local/bin/goose
  And the installation completes without errors

Scenario: Goose is accessible in PATH
  Given the container is running
  When I execute goose --version
  Then I see the Goose version number
  And the command exits with status 0

Scenario: Goose can be updated
  Given Goose is installed in the container
  When I execute goose update
  Then the command checks for updates
  And completes without errors
```

---

## REQ-003: Workspace Volume Mounting

### Motivation
Mount the user's current working directory (where goose-run is invoked) as /workspace in the container. This limits the blast radius of the LLM agent framework to only the Docker container and the mounted project directory, preventing unauthorised access to other parts of the host filesystem. The container should have no access to files outside the mounted directory. Changes made within the container persist to the host and vice versa.

### In Scope
- Dynamically mount $(PWD) (current working directory) to /workspace
- Set /workspace as the container's working directory
- Ensure container has no access to host filesystem outside the mounted directory
- Support bidirectional file synchronisation between host and container
- Ensure file permissions work correctly across host and container

### Acceptance Criteria

```gherkin
Scenario: Current directory is dynamically mounted as workspace
  Given I run goose-run from /path/to/my-project
  When the container starts
  Then /workspace in container maps to /path/to/my-project on host
  And the container working directory is /workspace
  And files in /path/to/my-project are accessible in /workspace
  And no other host directories are accessible

Scenario: Subdirectory mounting works
  Given I run goose-run from /path/to/my-project/subdir
  When the container starts
  Then /workspace in container maps to /path/to/my-project/subdir on host
  And only files in subdir and below are accessible to Goose

Scenario: Filesystem isolation is enforced
  Given the container is running
  When Goose attempts to access files outside /workspace
  Then access is denied
  And only the mounted project directory is accessible

Scenario: Files synchronise bidirectionally
  Given the container is running with mounted workspace
  When I create a file in the container at /workspace/test.txt
  Then the file appears on the host in the original directory
  When I create a file on the host in the mounted directory
  Then the file appears in the container at /workspace

Scenario: File permissions are preserved
  Given a file is created in mounted workspace
  When I check file ownership from host and container
  Then the file is readable and writable from both
  And permissions are preserved across host and container
```

---

## REQ-004: Environment Variable Configuration

### Motivation
Allow configuration of API keys via environment variables for security and flexibility. Sensitive data should never be baked into the image. Goose configuration is hardcoded to /workspace/.goose to maintain security isolation and predictability.

### In Scope
- Support OPENROUTER_API_KEY environment variable
- Hardcode GOOSE_HOME to /workspace/.goose (not configurable)
- Document required environment variables

### Acceptance Criteria

```gherkin
Scenario: OpenRouter API key is accessible
  Given the container is started with OPENROUTER_API_KEY set
  When I check the environment variable
  Then OPENROUTER_API_KEY contains the provided value

Scenario: Goose home directory is hardcoded
  Given the container is running
  When I check the GOOSE_HOME environment variable
  Then it is set to /workspace/.goose
  And Goose stores all configuration in /workspace/.goose
```

---

## REQ-005: Goose Configuration Initialisation

### Motivation
Initialise Goose configuration directory and context files in the workspace on first run. This provides a ready-to-use environment whilst keeping configuration within the mounted project directory for persistence and portability.

### In Scope
- Create /workspace/.goose directory if it doesn't exist
- Create placeholder .goosehints file if it doesn't exist
- Document purpose and usage of each file
- Ensure all configuration persists across container restarts

### Acceptance Criteria

```gherkin
Scenario: Goose configuration directory is initialised
  Given the container starts for the first time in a project
  When I inspect /workspace
  Then /workspace/.goose directory exists
  And Goose stores all configuration in /workspace/.goose

Scenario: Context files are initialised if missing
  Given the container starts in a project without context files
  When I inspect /workspace
  Then /workspace/.goosehints file exists with placeholder content

Scenario: Existing configuration is preserved
  Given /workspace/.goose already exists with user configuration
  When I restart the container
  Then existing configuration is preserved
  And no files are overwritten

Scenario: Configuration persists across container restarts
  Given I modify Goose configuration in /workspace/.goose
  When I restart the container from the same directory
  Then my configuration changes are still present
```

---

## REQ-006: Startup Information

### Motivation
Provide clear feedback when entering the container about what is configured, what tools are available, and how to get started. This improves user experience and reduces troubleshooting time.

### In Scope
- Display welcome message on container start
- Show Goose version information
- Indicate which environment variables are configured
- List available commands
- Show status of context files

### Acceptance Criteria

```gherkin
Scenario: Welcome message displays
  Given the container starts
  When the entrypoint script runs
  Then I see "Goose Development Environment Ready"
  And I see the Goose version number
  And I see available commands

Scenario: Configuration status is shown
  Given OPENROUTER_API_KEY is set
  When the container starts
  Then I see "OpenRouter API configured"

Scenario: Context files are detected
  Given .goosehints exists in workspace
  When the container starts
  Then I see "✓ .goosehints file detected"
```

---

## REQ-007: Resource Management

### Motivation
Control container resource usage to prevent the container from consuming excessive system resources while still providing adequate performance for AI-assisted development work.

### In Scope
- Set memory limits in docker-compose.yml
- Set memory reservations
- Document resource requirements
- Make resources easily adjustable

### Acceptance Criteria

```gherkin
Scenario: Memory limits are enforced
  Given docker-compose.yml has memory limit of 16G
  When the container is running
  Then the container cannot exceed 16GB of memory

Scenario: Memory is reserved
  Given docker-compose.yml has memory reservation of 4G
  When the container starts
  Then 4GB of memory is reserved for the container

Scenario: Resources are configurable
  Given I want to change memory limits
  When I edit docker-compose.yml
  Then I can adjust memory limits without modifying other configuration
```

---

## REQ-008: Network Configuration

### Motivation
Allow the container to access OpenRouter API and other external services while maintaining security isolation. Support future integration with local Ollama/Qwen models running on host.

### In Scope
- Enable external network access for API calls
- Document network configuration
- Support host.docker.internal for local model access
- Maintain security boundaries

### Acceptance Criteria

```gherkin
Scenario: External API access works
  Given the container is running
  And OPENROUTER_API_KEY is configured
  When Goose makes an API call to OpenRouter
  Then the request succeeds

Scenarino: Host access is configurable
  Given extra_hosts is configured with host.docker.internal
  When I access host.docker.internal from container
  Then I can reach services on the host machie

Scenario: Network isolation is maintained
  Given the container is running
  When I attempt to access host filesystem
  Then access is denied outside mounted volumes
```

---

## REQ-009: Container Lifecycle Management

### Motivation
Provide simple, documented commands for common container operations. Users should be able to start, stop, rebuild, and troubleshoot the container without deep Docker knowledge.

### In Scope
- Create setup script that generates .env file template
- Document start/stop commands
- Document rebuild process
- Document troubleshooting steps

### Acceptance Criteria

```gherkin
Scenario: Setup script initialises environment
  Given I run ./setup.sh
  When the script completes
  Then .env file exists
  And I see next steps instructions

Scenario: Container starts easily
  Given setup is complete
  When I run docker-compose up -d
  Then the container starts in detached mode
  And I see "container started" message

Scenario: Container can be rebuilt
  Given the Dockerfile has changed
  When I run docker-compose up -d --build
  Then the container rebuilds with new configuration
  And starts successfully

Scenario: User can enter running container
  Given the container is running
  When I run docker-compose exec goose-dev bash
  Then I have a shell inside the container
  And I see the welcome message
```

---

## REQ-010: Local Installation and CLI Tool

### Motivation
Provide a simple installation process that makes `goose-run` available as a global command on the host system. Users should be able to run `goose-run` from any project directory without needing to navigate to the goose-run repository or manage Docker commands directly.

### In Scope
- Install `goose-run` script to a standard location in user's PATH
- Support installation to `~/.local/bin` (preferred) or `/usr/local/bin` (with sudo)
- Provide uninstall capability
- Verify Docker and docker-compose are available
- Document installation requirements

### Acceptance Criteria

```gherkin
Scenario: Installation script installs goose-run command
  Given I run ./setup.sh
  When the installation completes
  Then goose-run is installed to ~/.local/bin/goose-run
  And ~/.local/bin is in my PATH
  And I can run goose-run from any directory

Scenario: Installation checks prerequisites
  Given I run ./setup.sh
  When the script checks system requirements
  Then it verifies Docker is installed
  And it verifies docker-compose is installed
  And it fails gracefully with clear error messages if dependencies are missing

Scenario: Installation is idempotent
  Given goose-run is already installed
  When I run ./setup.sh again
  Then the installation succeeds
  And the existing installation is updated

Scenario: Uninstallation removes goose-run
  Given goose-run is installed
  When I run ./setup.sh --uninstall
  Then goose-run is removed from ~/.local/bin
  And I see a confirmation message

Scenario: goose-run command is globally accessible
  Given installation is complete
  When I navigate to any directory
  And I run goose-run --version
  Then the command executes successfully
```

---

## REQ-011: Transparent Command Pass-through

### Motivation
Make `goose-run` behave as a transparent wrapper around the containerised Goose installation. Users should be able to use `goose-run` exactly as they would use `goose` natively, with all commands, arguments, and interactive features working seamlessly whilst maintaining security isolation.

### In Scope
- Pass all command-line arguments to `goose` executable inside container
- Read `OPENROUTER_API_KEY` from host environment variables
- Automatically start container if not running
- Attach to existing container if already running
- Support interactive commands (e.g., `goose session`, `goose configure`)
- Preserve exit codes from goose commands
- Stream stdout/stderr from container to host

### Acceptance Criteria

```gherkin
Scenario: Arguments are passed through to goose
  Given I have OPENROUTER_API_KEY set in my environment
  When I run goose-run session -r
  Then goose session -r executes inside the container
  And I see the interactive Goose session
  And the session has access to /workspace files

Scenario: Configuration commands work
  Given I run goose-run configure
  When I interactively configure Goose settings
  Then configuration is saved to /workspace/.goose
  And the configuration persists after container restart

Scenario: Version and help commands work
  Given I run goose-run --version
  Then I see the Goose version from inside the container
  When I run goose-run --help
  Then I see Goose help information

Scenario: Environment variables are passed from host
  Given OPENROUTER_API_KEY is set to "test-key-123" on host
  When I run goose-run from any directory
  Then the container receives OPENROUTER_API_KEY="test-key-123"
  And Goose can authenticate with OpenRouter

Scenario: Container lifecycle is managed automatically
  Given no container is running
  When I run goose-run session
  Then the container starts automatically
  And goose session executes
  When I run goose-run --version in another terminal
  Then it attaches to the existing container
  And does not start a new container

Scenario: Exit codes are preserved
  Given I run goose-run with an invalid command
  When the command fails inside the container
  Then goose-run exits with the same non-zero exit code
  And I can check $? to see the failure

Scenario: Current directory is mounted as workspace
  Given I am in /home/user/my-project
  When I run goose-run session
  Then /home/user/my-project is mounted as /workspace in container
  And Goose has access to my project files
  When I navigate to /home/user/another-project
  And I run goose-run session
  Then /home/user/another-project is mounted as /workspace
```

---

## REQ-012: Container Lifecycle Control Commands

### Motivation
Provide explicit commands for managing the container lifecycle. Whilst `goose-run` automatically starts the container when needed, users should have control over stopping and restarting the container to free resources, apply updates, or troubleshoot issues. Exiting a Goose session should not stop the container, allowing for quick reconnection.

### In Scope
- `goose-run stop` command to stop the running container
- `goose-run restart` command to restart the container
- `goose-run status` command to check if container is running
- Normal session exit leaves container running
- Clear feedback on container state changes

### Acceptance Criteria

```gherkin
Scenario: Exiting Goose session leaves container running
  Given I run goose-run session
  And the container is running
  When I exit the Goose session
  Then the Goose session terminates
  But the container remains running
  And I can run goose-run session again to reconnect quickly

Scenario: Stop command stops the container
  Given the container is running
  When I run goose-run stop
  Then the container stops gracefully
  And I see "Container stopped" message
  And docker ps shows no goose container running

Scenario: Stop command on stopped container is safe
  Given the container is not running
  When I run goose-run stop
  Then the command succeeds
  And I see "Container already stopped" message
  And no error is raised

Scenario: Restart command restarts the container
  Given the container is running
  When I run goose-run restart
  Then the container stops gracefully
  And the container starts again
  And I see "Container restarted" message
  And the container has fresh state

Scenario: Restart is useful after Goose updates
  Given I have updated Goose in the container image
  And the container is running with old Goose version
  When I run goose-run restart
  Then the container restarts with new Goose version
  And goose-run --version shows the updated version

Scenario: Status command shows container state
  Given the container is running
  When I run goose-run status
  Then I see "Container: running"
  And I see container uptime information
  When I stop the container
  And I run goose-run status
  Then I see "Container: stopped"

Scenario: Multiple sessions can coexist
  Given I run goose-run session in terminal 1
  When I run goose-run session in terminal 2
  Then both terminals connect to the same container
  And I can interact with Goose from both terminals
  When I exit from terminal 1
  Then terminal 2 session continues working
  And the container remains running
```
