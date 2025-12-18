# REQ-012: Container Lifecycle Control Commands

## Motivation
Provide explicit commands for managing the container lifecycle. Whilst `goose-run` automatically starts the container when needed, users should have control over stopping and restarting the container to free resources, apply updates, or troubleshoot issues. Exiting a Goose session should not stop the container, allowing for quick reconnection.

## In Scope
- `goose-run stop` command to stop the running container
- `goose-run restart` command to restart the container
- `goose-run status` command to check if container is running
- Normal session exit leaves container running
- Clear feedback on container state changes

## Acceptance Criteria

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
