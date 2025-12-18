# REQ-006: Startup Information

## Motivation
Provide clear feedback when entering the container about what is configured, what tools are available, and how to get started. This improves user experience and reduces troubleshooting time.

## In Scope
- Display welcome message on container start
- Show Goose version information
- Indicate which environment variables are configured
- List available commands
- Show status of context files

## Acceptance Criteria

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
