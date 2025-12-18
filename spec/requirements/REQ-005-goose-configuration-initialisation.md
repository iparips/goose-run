# REQ-005: Goose Configuration Initialisation

## Motivation
Initialise Goose configuration directory and context files in the workspace on first run. This provides a ready-to-use environment whilst keeping configuration within the mounted project directory for persistence and portability.

## In Scope
- Create /workspace/.goose directory if it doesn't exist
- Create placeholder .goosehints file if it doesn't exist
- Document purpose and usage of each file
- Ensure all configuration persists across container restarts

## Acceptance Criteria

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
