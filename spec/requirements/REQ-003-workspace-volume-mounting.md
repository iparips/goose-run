# REQ-003: Workspace Volume Mounting

## Motivation
Mount the user's current working directory (where goose-run is invoked) as /workspace in the container. This limits the blast radius of the LLM agent framework to only the Docker container and the mounted project directory, preventing unauthorised access to other parts of the host filesystem. The container should have no access to files outside the mounted directory. Changes made within the container persist to the host and vice versa.

## In Scope
- Dynamically mount $(PWD) (current working directory) to /workspace
- Set /workspace as the container's working directory
- Ensure container has no access to host filesystem outside the mounted directory
- Support bidirectional file synchronisation between host and container
- Ensure file permissions work correctly across host and container

## Acceptance Criteria

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
