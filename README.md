# Auto Pull Request with Merge

This GitHub Action automates the process of creating a pull request with an empty commit and optionally merging it. This can be useful for triggering workflows or deployments on specific branches at regular intervals.

[![Auto Pull Request](https://github.com/themuuln/auto-pull-request/actions/workflows/auto-pull-request.yml/badge.svg)](https://github.com/themuuln/auto-pull-request/actions/workflows/auto-pull-request.yml)

## Usage

### Schedule

The action is scheduled to run every 5 minutes. You can adjust the schedule based on your requirements. The current schedule is set as follows:

```yaml
on:
  schedule:
    - cron: "*/5 * * * *"
```

### Workflow

Create a `.github/workflows/auto-pull-request.yml` file in your repository with the following content:

```yaml
name: Auto Pull Request with Merge

on:
  schedule:
    - cron: "*/5 * * * *"

jobs:
  auto_pull_request:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set Git User Name and Email
        run: |
          git config user.name "themuuln"  # Replace with your GitHub username
          git config user.email "zerone.offical@gmail.com"  # Replace with your GitHub email

      - name: Create Empty Pull Request
        run: |
          cd $GITHUB_WORKSPACE
          branch="main"  # Change this to the target branch

          # Generate a unique branch name
          branch_name="auto-pull-request-$(date +%s)"

          # Create a new branch
          git checkout -b $branch_name

          # Create an empty commit to trigger a pull request
          git commit --allow-empty -m "Auto Pull Request Trigger"

          # Push the new branch to the repository
          git push origin $branch_name

          # Create the pull request using your GitHub credentials
          pr_url=$(curl -X POST -u "YourGitHubUsername:$PERSONAL_ACCESS_TOKEN" -d '{
            "title": "Automated Pull Request",
            "head": "'$branch_name'",
            "base": "'$branch'"
          }' "https://api.github.com/repos/USERNAME/REPO_NAME/pulls")

          # Merge the pull request (if needed) using your GitHub credentials
          curl -X PUT -u "YourGitHubUsername:$PERSONAL_ACCESS_TOKEN" "$pr_url/merge"
        env:
          PERSONAL_ACCESS_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

Make sure to replace the placeholder values (`YourGitHubUsername`, `USERNAME`, `REPO_NAME`) with your actual GitHub username and repository details.

### Personal Access Token

To securely use your GitHub credentials, create a personal access token and add it as a secret named `PERSONAL_ACCESS_TOKEN` in your GitHub repository settings.

Now, every 5 minutes, this workflow will create an empty pull request on the specified branch, triggering any associated workflows or deployments. Optionally, it can merge the pull request based on your configuration. Adjust the branch names and other details as needed for your specific use case.
