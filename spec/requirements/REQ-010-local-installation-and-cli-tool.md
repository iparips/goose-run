# REQ-010: Local Installation and CLI Tool

## Motivation
Provide a simple installation process that makes `goose-run` available as a global command on the host system. Users should be able to run `goose-run` from any project directory without needing to navigate to the goose-run repository or manage Docker commands directly.

## In Scope
- Install `goose-run` script to a standard location in user's PATH
- Support installation to `~/.local/bin` (preferred) or `/usr/local/bin` (with sudo)
- Provide uninstall capability
- Verify Docker and docker-compose are available
- Document installation requirements

## Acceptance Criteria

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
