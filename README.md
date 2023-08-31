# workflow-script-injection

The [Check issue title](.github/workflows/check-issue-title.yml) workflow simple checks if the title of the workflow begins with `octocat`. If so the workflow succeeds. If not, the workflow fails.

This workflow is vulnerable to script injection. Let's find out why.
