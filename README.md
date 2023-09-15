# workflow-script-injection

The [Check issue title](.github/workflows/check-issue-title.yml) workflow simply checks if the title of the workflow begins with `octocat`. If so, the workflow succeeds. If not, the workflow fails.

The [Check issue comment](.github/workflows/check-issue-comment.yml) workflow simply checks if the issue comment begins with `octocat`. If so, the workflow succeeds. If not, the workflow fails.

These workflows are vulnerable to script injection. Let's find out why.
