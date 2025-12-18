# REQ-011: Transparent Command Pass-through

## Motivation
Make `goose-run` behave as a transparent wrapper around the containerised Goose installation. Users should be able to use `goose-run` exactly as they would use `goose` natively, with all commands, arguments, and interactive features working seamlessly whilst maintaining security isolation.

## In Scope
- Pass all command-line arguments to `goose` executable inside container
- Read `OPENROUTER_API_KEY` from host environment variables
- Automatically start container if not running
- Attach to existing container if already running
- Support interactive commands (e.g., `goose session`, `goose configure`)
- Preserve exit codes from goose commands
- Stream stdout/stderr from container to host

## Acceptance Criteria

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
