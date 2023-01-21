# GitHub Actions Workflow Configuration

This template contains 2 workflows:
1. `main.yml` - This workflow is triggered on pull requests to the `main`, `master` or `develop` branches OR on pushes to `main`, `master` or `develop` branches.
2. `release.yml` - This workflow is triggered on pushes to `main` or `master` branches.

## Configuration
For the workflows to run, you need to configure some secrets in your repository settings as well as make some string replacements in the workflow files.

### Environment Variables
In both `main.yml` and `release.yml`, you need to inspect environment variable definitions and insert your own values.

In `main.yml`:
- The addon name (this will affect paths)

In `release.yml`:

- The addon name (this will affect paths)
- Godot Asset Lib and
- Itch.io,

respectively.

### Secrets
#### GITHUB_TOKEN
[`GITHUB_TOKEN`](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) is an automatically injected secret, which is only valid for the time of the current action run. However, in it's default configuration, there are no write permissions. This is why there is a configuration in the `release.yml` configuration to assign this permission for the scope of this workflow.

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

breaking: Removed legacy ClientAPI
```
