# Reuseable CI/CD

## Github Actions

Composite actions are located in the .github/actions directory. The following actions are supported:

- test
- publish_sdk
- build
- update_deployment

### How to use

To add a step in your workflow, use the following format:

- `uses: megatron-global/ci-cd/.github/actions/{action}@main`

Replace {action} with the desired action name.

### Publish SDK Action

When using the publish_sdk action, the following input variables must be provided:

#### Input Variables:
    - aws-secret-key
    - aws-access-key-id
    - gitlab_repository
    - gitlab_username
    - gitlab_password

### Build Action

We need to pass these input variables:

#### Environment Variables: 
    - PROJECT
    - ECR_REPOSITORY

#### Input Variables:
    - aws-secret-key
    - aws-access-key-id
    - aws-region


### Update Deployment Action

#### Environment Variables: 
    - ENV
    - PROJECT
    - HELM_CHART_NAME
    - HELM_DEPLOYMENT_DIRECTORY
    - REPOSITORY_URL
    - REPOSITORY_NAME

```yaml
name: CI/CD
on: push

jobs:
  test:
    runs-on: self-hosted
    steps:

      - uses: actions/checkout@v4
      - name: Run e2e test
        uses: megatron-global/ci-cd/.github/actions/test@main

  publish-sdk:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Publish SDK
        uses: megatron-global/ci-cd/.github/actions/publish_sdk@main
        with:
          aws-secret-key: ${{ env.AWS_SECRET_KEY }}
          aws-access-key-id: ${{ env.AWS_ACCESS_KEY_ID }}
          gitlab_repository: ${{ env.SDK_GITLAB_REPOSITORY }}
          gitlab_username: ${{ env.SDK_GITLAB_USERNAME }}
          gitlab_password: ${{ env.SDK_GITLAB_PASSWORD }}

  build:
    runs-on: self-hosted
    env:
      PROJECT: {project-name}
      ECR_REPOSITORY: {project-name}
    steps:
      - uses: actions/checkout@v4
      - name: Build docker image
        uses: megatron-global/ci-cd/.github/actions/build_image@main
        with:
          aws-secret-key: ${{ env.AWS_SECRET_KEY }}
          aws-access-key-id: ${{ env.AWS_ACCESS_KEY_ID }}
          aws-region: ${{ env.AWS_REGION }}

  update-deployment:
    needs: [build]
    runs-on: self-hosted 
    env:
      ENV: dev|uat|prod
      REPOSITORY_URL: {deployment-repository-url}
      REPOSITORY_NAME: {deployment-repository-name}
      HELM_DEPLOYMENT_DIRECTORY: {helm-deployment-directory}
      PROJECT: {project-name}
      HELM_CHART_NAME: {helm-chart-name}
    steps:
      - uses: actions/checkout@v4 
      - name: Update deployment
        uses: megatron-global/ci-cd/.github/actions/update_deployment@main
```
