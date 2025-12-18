# REQ-008: Network Configuration

## Motivation
Allow the container to access OpenRouter API and other external services while maintaining security isolation. Support future integration with local Ollama/Qwen models running on host.

## In Scope
- Enable external network access for API calls
- Document network configuration
- Support host.docker.internal for local model access
- Maintain security boundaries

## Acceptance Criteria

```gherkin
Scenario: External API access works
  Given the container is running
  And OPENROUTER_API_KEY is configured
  When Goose makes an API call to OpenRouter
  Then the request succeeds

Scenario: Host access is configurable
  Given extra_hosts is configured with host.docker.internal
  When I access host.docker.internal from container
  Then I can reach services on the host machine

Scenario: Network isolation is maintained
  Given the container is running
  When I attempt to access host filesystem
  Then access is denied outside mounted volumes
```
