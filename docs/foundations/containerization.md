# Containerization

All services are containerized using Docker.

## What was done

Services are packaged as Docker images using multi-stage builds. Build dependencies stay in earlier stages, and only runtime artifacts make it to the final image. This keeps images small and deployment-ready.

## Why this matters

Smaller images mean faster pulls, quicker startup times, and less wasted cluster resources. When running multiple microservices, these small gains add up across the entire deployment.

## Validation

Images were built successfully and are ready for deployment.

![Docker Build](../../screenshots/foundations/docker-build.png)
