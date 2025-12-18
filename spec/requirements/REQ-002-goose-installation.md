# REQ-002: Goose Installation

## Motivation
Install the latest stable version of Goose using the official installation method. Avoid Python dependencies since Goose is a Rust binary, reducing container size and complexity.

## In Scope
- Download and install Goose using official GitHub installation script
- Configure Goose to skip interactive setup during build
- Add Goose binary to system PATH
- Enable version updates via `goose update` command

## Acceptance Criteria

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
