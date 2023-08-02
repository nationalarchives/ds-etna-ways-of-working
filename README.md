# Proposed Ways of Working for ETNA

## Environments

- Production/live
- Staging/pre-production
- Development
- Test
- Local

| Environment | Application              | Version                       | Database                            | Server                                |
| ----------- | ------------------------ | ----------------------------- | ----------------------------------- | ------------------------------------- |
| Production  | Docker image from GitHub | Latest deployed release       | Live Postgres database              | Production EC2 instance behind nginx  |
| Staging     | Docker image from GitHub | Next release                  | Copy of live database               | Staging EC2 instance behind nginx     |
| Development | Docker image from GitHub | In line with `main` branch    | Copy of live database               | Development EC2 instance behind nginx |
| Test        | Docker image from GitHub | In line with feature branches | Copy of live database inside Docker | Development EC2 instance              |
| Local       | Local Docker image       | -                             | Copy of live database inside Docker | Docker Desktop                        |

## Promotion process

![Promotion process diagram](./promotion-process.png)

### Local

- Anything goes

### Local → Test

1. Create new branch for feature (`feature/*`)
1. Test environment automatically created through GitHub Actions
1. Docker image created and added to test environment
1. Test latest features in an AWS environment
1. Push changes to rebuild image and update hosted application

### Test

- Test applciation is accessible to all
- Postgres database created as a Docker image
- Feature branch can be deleted at any point and environment will also be deleted

### Test → Development

1. Open PR from feature branch (`feature/*`) to working branch `main`(?)
1. Reviewer can use deployed test environment to check changes
1. Reviewer approves changes
1. Merge after review
1. Docker image (`develop`) updated and deployed to development environment
1. Latest Docker image is also automatically created (`latest`)

### Development

- Development applciation is accessible to all
- Follows working branch
- Working branch is protected
- More CI needed for E2E and acceptance tests
- **Working branch should always be deployable**

### Development → Staging

1. Cut a release branch `release/*` from working branch using semver but don't include the build version (e.g. `release/7.4`)
1. Others can carry on pushing to working branch
1. Each push to release branch will create a new tag using sequential number (e.g. `v7.4.32749`)
1. Create a release using the tag you want to deploy
1. Versioned Docker image is automatically created using the release tag

### Staging

- For stakeholders to test and signoff latest features before go live
- Database is copied from production
- Try anything

### Staging → Production

1. [TODO]

### Hotfix process

When we need to make a fix to production code but there is already changes in the trunk that we don't want to release.

1. Create a branch from the release you want to hotfix (e.g. from `release/7.4`)
1. Apply the fix
1. Open a PR back into the release branch
1. Reviewer approves changes
1. Merge after review
1. Versioned Docker image is automatically created using sequential number (e.g. `7.4.34921`)
1. Create a release using the new tag (Docker will build)
1. [TODO] - same process as staging → production
1. Open a PR to duplicate the changes back into the working branch

[Hotfix process diagram](https://www.mermaidchart.com/app/projects/1236b2e9-bed7-46dd-beb3-98a52fc4c209/diagrams/6bac87a1-ca63-438d-81dc-9128aa2f0baa/version/v0.1/view)

## Database

```mermaid
sequenceDiagram
    Production->>Store: Nightly backup
    Note over Store: Full database snapshot
    Store->>Staging: Manual process
    Store->>Store: Strip sensitive data
    Note over Store: Stripped database snapshot
    Store->>Development: Automated nightly process
    Store->>Test: On environment creation
    Store->>Local: Manual process
```
