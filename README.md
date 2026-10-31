# docker

Reusable Docker images for web apps and CI, built on official base images
and published to Docker Hub.

| Folder | Docker Hub | Contents |
|---|---|---|
| [php](php) | `tuanhmt/php` | PHP-FPM and PHP CLI images for PHP apps |

## How it is organized

Each image has its own folder with a Dockerfile and a README, and its own
GitHub Actions workflow in `.github/workflows/<folder>.yml`. A workflow runs
only when its folder changes, plus once a week to pick up security updates
from the base images.

Workflows push to Docker Hub from the `master` branch only. Pull requests
build and test without pushing.

## Adding an image

1. Create a folder with a Dockerfile and a README.
2. Copy `.github/workflows/php.yml`, then change the name, the paths and
   the build and test steps.
3. Add a row to the table above.

## Repository secrets

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub user name, also the image namespace |
| `DOCKERHUB_TOKEN` | Docker Hub access token with read and write access |
