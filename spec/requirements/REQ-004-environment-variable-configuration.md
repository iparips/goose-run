# REQ-004: Environment Variable Configuration

## Motivation
Allow configuration of API keys via environment variables for security and flexibility. Sensitive data should never be baked into the image. Goose configuration is hardcoded to /workspace/.goose to maintain security isolation and predictability.

## In Scope
- Support OPENROUTER_API_KEY environment variable
- Hardcode GOOSE_HOME to /workspace/.goose (not configurable)
- Document required environment variables

## Acceptance Criteria

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
