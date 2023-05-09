# Shared Workflows
Collection of github action workflow templates

## Dotnet Test and Publish Results
This workflow will build and run all test in a dotnet solution. It will also publish the results of the tests.
It will fail if build fails or any test does.

### Parameters
- **project_path** - Path to the directory where the solution file is located
- **dotnet-version** - The dotnet version used in the project
### Secrets
- **nuget_username** - Username for package registry. This parameters is defined as a global organization variable named _NUGET_USERNAME_  
- **nuget_pat** - PAT for package registry. This parameters is defined as a global organization variable named _NUGET_PAT_

```yaml
    name: Build & Test Solution
    uses: cloudfactorydk/shared-workflows/.github/workflows/dotnet-test.yml@main
    with:
      project_path: .
      dotnet-version: 7.0.x
    secrets:
      nuget_username: ${{ secrets.NUGET_USERNAME }}
      nuget_pat: ${{ secrets.NUGET_PAT }}
```

## Create and Publish Docker Container
This workflow will create and build a docker container.

### Parameters
- **project_path** - Path typical the path to the directory where the solution file is located.
- **dockerfile_path** - Path to the docker file
- **docker_image_name** - The image name of the docker container
### Secrets
- **gh_token** - Github token 
- **nuget_username** - Username for package registry. This parameters is defined as a global organization variable named _NUGET_USERNAME_
- **nuget_pat** - PAT for package registry. This parameters is defined as a global organization variable named _NUGET_PAT_

```yaml
    name: Build Docker Containers
    uses: cloudfactorydk/shared-workflows/.github/workflows/dotnet-build-test-publish-docker.yml@main
    with:
      project_path: .
      dockerfile_path: ${{ matrix.dockerfile }}
      docker_image_name: ${{ matrix.image_name }}
    secrets:
      gh_token: ${{ secrets.GITHUB_TOKEN }}
      nuget_username: ${{ secrets.NUGET_USERNAME }}
      nuget_pat: ${{ secrets.NUGET_PAT }}
```

## Example of a cobined workflow

```yaml
name: Publish Docker Containers

on:
  push:
    branches:
      - development
      - feature/**
      - release-*
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+"
      - "v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+"
      - "v[0-9]+.[0-9]+.[0-9]+-alpha[0-9]+"
    workflow_dispatch:

permissions:
  contents: read
  issues: read
  checks: write
  pull-requests: write
  packages: write

jobs:
  build-test:
    name: Build & Test Solution
    uses: cloudfactorydk/shared-workflows/.github/workflows/dotnet-test.yml@main
    with:
      project_path: .
      dotnet-version: 6.0.x
    secrets:
      nuget_username: ${{ secrets.NUGET_USERNAME }}
      nuget_pat: ${{ secrets.NUGET_PAT }}

  build-docker-containers:
    name: Build Docker Containers
    needs: build-test
    
    strategy:
      fail-fast: true
      matrix:
        include:
          - image_name: cf.catalogue.rest
            dockerfile: ./CF.Catalogue/CatalogueRestApi/Dockerfile
          - image_name: cf.cataloguesignalservice.services
            dockerfile: ./CF.Catalogue/CatalogueSignalService/Dockerfile
    
    uses: cloudfactorydk/shared-workflows/.github/workflows/dotnet-build-test-publish-docker.yml@main
    with:
      project_path: .
      dockerfile_path: ${{ matrix.dockerfile }}
      docker_image_name: ${{ matrix.image_name }}
    secrets:
      gh_token: ${{ secrets.GITHUB_TOKEN }}
      nuget_username: ${{ secrets.NUGET_USERNAME }}
      nuget_pat: ${{ secrets.NUGET_PAT }}
```