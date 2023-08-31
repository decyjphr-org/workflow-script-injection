# workflow-script-injection

The [Check issue title](.github/workflows/check-issue-title.yml) workflow simple checks if the title of the workflow begins with `octocat`. If so the workflow succeeds. If not, the workflow fails.

This workflow is vulnerable to script injection. Let's find out why.

## Exercise 1
The [Check issue title](.github/workflows/check-issue-title.yml) workflow uses the issue title in the `run` command as follows:
```
title="${{ github.event.issue.title }}"
```
This provides an opportunity for an attacker to exploit the workflow with an issue titled `octocat"; ls -l $GITHUB_WORKSPACE"`. Using this title, the script looks like the following...
```
title="octocat"; ls -l $GITHUB_WORKSPACE""
```

Let's create a new issue with this title and see what happens. We observe that the workflow runs the command `ls -l $GITHUB_WORKSPACE`!
