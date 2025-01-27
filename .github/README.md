﻿# GitHub Actions Workflow Configuration

This template contains 2 workflows:
1. `main.yml` - This workflow is triggered on pull requests to the `main`, `master` or `develop` branches OR on pushes to `main`, `master` or `develop` branches.
2. `release.yml` - This workflow is triggered on pushes to `main` or `master` branches.

## Configuration
For the workflows to run, you need to configure some secrets in your repository settings as well as make some string replacements in the workflow files.

### Environment Variables
In `release.yml`, you need to inspect the environment variable definitions and insert your own values for Godot Asset Lib and Itch.io, respectively.

### String Replacements
In both `main.yml` and `release.yml`, you need to replace, just with the rest of this template repository, replace all occurrences of `ADDON_NAME` with the name of your addon! Please make sure there are no spaces, and only alphanumeric values, dashes and underscores in this sting!.

### Secrets
#### PAT
Because we're committing back from the workflow to the repo, you need to provide a "Personal Access Token" or "PAT" to the workflow. This is a token that allows the workflow to commit back to the repo. You can create a PAT in your GitHub settings. [See the documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
This PAT needs to have the `repo` scope.

Once you have created this PAT, you need to add it to the repository secrets as `PAT`.

##### Why not use `GITHUB_TOKEN`?
`GITHUB_TOKEN` does not allow commits back to the same repo for security reasons. Hence, we need this specific PAT to allow committing back.

#### Godot Asset Library
If you want to publish to the Godot Asset Library, you need to add the following secrets to your repository:

* `ASSET_LIB_PASSWORD` - The password for your Godot Asset Library account

#### Itch.io
If you want to publish to Itch.io, you need to add the following secrets to your repository:

* `ITCHIO_SECRET` - This is the secret key for your Itch.io page. You can find it in your addon's settings on Itch.io.

There are more options to this. Please check out the [respective action's documentation](https://github.com/KikimoraGames/itch-publish).



## How does it work?
### `main.yml`
This is the main workflow that runs on pull requests and pushes to the `main`, `master` or `develop` branches. It just does a shallow git checkout (the latest revision) and runs GUT tests.


### `release.yml`
This workflow is triggered on pushes to `main` or `master` branches. It does a full git checkout (all revisions).
It is self-recursive!

#### First Run
In the first run, it will run the `prerelease` job, which will determine if and how to bump the version.
If there is a version bump determined, it will
* generate and update `CHANGELOG.md` (or create it, if it does not exist)
* update `plugin.cfg` with the new version
* Commit the changes back to the branch

#### Second Run
On the second run, it will skip the `prerelease` job and run the `release` job instead.
This will:

* Tag the release commit
* Upload a release artifact for later use
* Create a GitHub release, with the changelog as the release notes

Once the release job was run, the workflow will then run two other jobs to publish to Itch.io and the Godot Asset Library.


## How to do a version bump?
`release.yml` will automatically determine if and how to bump the version using the [Git Hub Action `mathieudutour/github-tag-action`](https://github.com/marketplace/actions/github-tag). It will do this by looking at the commit messages of the commits that are on the branch.

If you look closely, `release.yml` defines the keywords that are used to determine the version bump with the `custom_release_rules`, but these are in addition to the default Angular Commit Message Convention (see below). The pattern is always:

  ```
  <keyword>:<release type>:<changelog_section>
  ```

where `release_type` is one of ``major``, ``minor`` or ``patch`` and `changelog_section` is a free-form string, which defines a header in the `CHANGELOG.md`. If left empty, it defaults to the Angular defaults. See [Angular Commit Message Convention for more info](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-changelog-angular)

### Examples

Appears under "Performance Improvements" header, and under "Breaking Changes" with the breaking change explanation:
```
perf: pooled HTTP connections

BREAKING CHANGE: Removed legacy ClientAPI
```
