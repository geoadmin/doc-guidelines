# Continuous Integration

Each service or productive piece of software should be automatically tested, before merged to `develop`. We use [AWS CodeBuild](https://docs.aws.amazon.com/fr_fr/codebuild/) for building/testing code. For every software we have two Codebuild projects

- One which is triggered by Pull Request CREATE and UPDATE event
  - This one lints, checks formatting, builds and test the code
- One which is triggered by Pull Request merge event
  - This one builds and releases the build artefact (release on S3 build artifact or ECR repository)

## Table of contents

- [Table of contents](#table-of-contents)
- [Guidelines](#guidelines)
- [Build Badge](#build-badge)

## Guidelines

- Each Codebuild project is managed via terraform in a dedicated repository
- `buildspec.yml` file is located within the terraform module of the Codebuild project which is in a private repository. This ensure security where a non authorized person cannot see and edit the buildspec and trigger a build on a public repository.
- Don't use secrets/password inside the buildspec but use AWS SSM [AWS System Manager (SSM)](https://docs.aws.amazon.com/systems-manager/index.html)
- Use build badge in the software github repository see [Build Badge](#build-badge)

## Build Badge

Build badge should be added to the top of the README.md file of the github project using the following template:

```md
| Branch | Status |
|--------|-----------|
| develop | ![Build Status](BADGE_LINK) |
| master | ![Build Status](BADGE_LINK) |
```

To get the badge login to the [AWS Console](https://console.aws.amazon.com) on the account managing Codebuild and navigate to the Codebuild project (make sure you are on the correct AWS zone `eu-central-1`). Then on the project main page click on `Copy badge URL`. This link looks as follow:

```bash
https://codebuild.eu-central-1.amazonaws.com/badges?uuid=eyJl...I6MX0%3D&branch=master
```

Change the branch at the end of the link accordingly.
