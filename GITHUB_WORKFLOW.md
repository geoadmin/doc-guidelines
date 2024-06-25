# Github Workflow for Geoadmin repositories

Most of the Geoadmin services repositories should be configured to use github workflow to automate the services release process.

We use github reusable workflow that are centralized in [geoadmin/.github](https://github.com/geoadmin/.github) repository. In the service repository we only need the github workflow to call the reusable workflow.

At best the workflows are automatically set during creation of the repository using terraform.

- [Setting up SemVer Workflow](#setting-up-semver-workflow)
- [Setting up Milestone Workflow](#setting-up-milestone-workflow)

## Setting up SemVer Workflow

For more information on this workflow see [SemVer 2.0 Version](VERSIONING_RELEASE.md#semver-20).

You can find the template that setup the workflow used by terraform here: [geoadmin/template-service-semver-public](https://github.com/geoadmin/template-service-semver-public)

:warning: Once the repository has been created and the workflows set, you need to create the initial tag `v0.0.0` due to a bug in the semver release workflow

```bash
git checkout master
git tag v0.0.0
git push origin v0.0.0
```

## Setting up Milestone Workflow

For more information on this workflow see [Milestone Version](VERSIONING_RELEASE.md#milestone-version).

You can find the template that setup the workflow used by terraform here: [geoadmin/template-service-milestone-public](https://github.com/geoadmin/template-service-milestone-public)
