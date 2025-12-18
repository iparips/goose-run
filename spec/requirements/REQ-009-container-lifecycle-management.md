# REQ-009: Container Lifecycle Management

## Motivation
Provide simple, documented commands for common container operations. Users should be able to start, stop, rebuild, and troubleshoot the container without deep Docker knowledge.

## In Scope
- Create setup script that generates .env file template
- Document start/stop commands
- Document rebuild process
- Document troubleshooting steps

## Acceptance Criteria

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
