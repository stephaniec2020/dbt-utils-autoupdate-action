
# Auto-update `dbt_utils`

This workflow (`auto-update-dbt-utils.yml`) keeps the `dbt_utils` dependency in `dbt/packages.yml` aligned with the latest GitHub release. It is designed for teams that want an automated Monday morning reminder when a new version ships but still prefer to review the pull request manually.

## How to set it up on your repository

1. Copy the `auto-update-dbt-utils.yml` workflow file into your repository’s `.github/workflows` directory.
2. Ensure your repository has a `develop` branch and a properly formatted `dbt/packages.yml` file.
3. Confirm your workflow token has `contents: write` and `pull-requests: write` permissions.
4. Adjust the workflow file as needed for your branch names or custom requirements.
5. Enable GitHub Actions in your repository settings.

For more details, see [How to use GitHub workflow templates](https://docs.github.com/en/actions/how-tos/write-workflows/use-workflow-templates).

## How it works
- Runs every Monday at 08:30 UTC (and can be dispatched manually).
- Checks out the `develop` branch with full history so it can create a new branch on top.
- Calls the GitHub API to find the newest `dbt_utils` release and reads the current version from `dbt/packages.yml` using `awk`.
- If the versions differ, rewrites the `version:` line under the `dbt-labs/dbt_utils` entry, creates a branch named `auto-update-dbt-utils-<latest-version>`, and commits the change.
- Pushes the branch back to the repository and posts a short summary in the workflow run output so a teammate can open the PR.

No commit or branch is created when the project is already on the most recent version.

## Prerequisites
- The project must contain `dbt/packages.yml` with a `dbt-labs/dbt_utils` entry formatted like:

	```yaml
	packages:
		- package: dbt-labs/dbt_utils
	      version: 1.1.0
	```
- The `develop` branch should exist, and the workflow token needs `contents: write` and `pull-requests: write` scopes (configured in the workflow file).
- Downstream tests or validations (e.g., `dbt deps` + CI) are expected to run outside of this workflow after the PR is opened.

## Manual triggering
From the **Actions** tab in GitHub, select **Auto-update dbt_utils version** and click **Run workflow**. This is useful if you want to cut an update mid-week instead of waiting for the next scheduled run.

## Customization ideas
- **Change the base branch:** update the `with.ref` value in the checkout step and the branch name logic if you use a different default branch.
- **Auto-open PRs:** add an `actions/github-script` step after the commit to file the pull request automatically.
- **Add validation:** insert extra steps (e.g., `dbt deps`, tests) before committing so that the update fails early if something breaks.
