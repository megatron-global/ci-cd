# Reuseable CI/CD

## Github Actions

Composite actions are located in the .github/actions directory. The following actions are supported:

- test
- publish_sdk
- build_image
- build_frontend_image
- update_deployment
- slack_start_deploying
- slack_finish_deploying
- discord_start_deploying
- discord_finish_deploying

### How to use

To add a step in your workflow, use the following format:

- `uses: rnyoo/pixer_github_action/.github/actions/{action}@main`

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
  start-deploying:
    needs: set-env
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: ${{ needs.set-env.outputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - name: Build docker image
        uses: rnyoo/pixer_github_action/.github/actions/slack_start_deploying@main
        with:
          slack-webhook-url: ${{ env.SLACK_WEBHOOK_URL }}

  test:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Run e2e test
        uses: rnyoo/pixer_github_action/.github/actions/test@main

  publish-sdk:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Publish SDK
        uses: rnyoo/pixer_github_action/.github/actions/publish_sdk@main
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
        uses: rnyoo/pixer_github_action/.github/actions/build_image@main
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
        uses: rnyoo/pixer_github_action/.github/actions/update_deployment@main

  finish-deploying:
    if: |
      github.event_name == 'push' ||
      github.event_name == 'pull_request' && (
        github.event.action == 'closed' &&
        github.event.pull_request.merged == true
      )
    needs: [...]
    env:
      ENVIRONMENT: ${{ needs.set-env.outputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - name: Notify deployment completion
        uses: rnyoo/pixer_github_action/.github/actions/slack_finish_deploying@main
        with:
          slack-webhook-url: ${{ env.SLACK_WEBHOOK_URL }}
          update-deployment-result: ${{ needs.update-deployment.result }}
```
