# REQ-001: Container Base System

## Motivation
Provide a lightweight, stable, and ARM64-optimised base system for running Goose in an isolated Docker environment. The base must be minimal to reduce attack surface and image size whilst supporting all necessary development tools.

## In Scope
- Use Debian Bookworm Slim as base image
- Install essential development tools (git, curl, build-essential)
- Configure for ARM64 architecture (Apple Silicon)
- Set up non-interactive installation mode

## Acceptance Criteria

```gherkin
Scenario: Container builds successfully
  Given the Dockerfile uses debian:bookworm-slim
  When I run docker build
  Then the build completes without errors
  And the final image size is under 200MB

Scenario: Essential tools are available
  Given the container is running
  When I execute git --version
  Then I see git version information
  When I execute curl --version
  Then I see curl version information

Scenario: ARM64 architecture is supported
  Given I am building on Apple Silicon
  When I inspect the built image
  Then the architecture is linux/arm64
```
