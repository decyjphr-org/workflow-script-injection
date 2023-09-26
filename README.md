<h1 align="center">Understanding the risk of script injection in GitHub Actions workflows</h1>
<h5 align="center">@robandpdx</h5>
<h5 align="center">@decyjphr</h5>

<p align="center">
  <a href="#mega-prerequisites">Prerequisites</a> •  
  <a href="#books-resources">Resources</a> •
  <a href="#learning-objectives">Learning Objectives</a>
</p>

In this workshop we will learn about the risk of script injection in GitHub Actions workflows, and how to mitigate that risk.  

- **Who is this for**: developers, devops engineers
- **What you'll learn**: the risk of Script Injections in GitHub Actions workflows, and how to migirate that risk, how to use CodeQL to detect vulnerabilities in Workflows, write custom queries to customize detection of issues in Workflows.
- **What you'll build**: Workflows that are not vulnerable to script injection attacks, CodeQL workflows to detect vulnerabilties, and custom CodeQL queries.

## Learning Objectives

In this workshop, you will:

  - learn about script injection vulnerabilities in GitHub actions workflows
  - learn how to mitigate script injection vulnerabilities in GitHub actions workflows
  - learn how Github Advanced Security can help you build secure GitHub actions workfows

## :mega: Prerequisites
Before joining the workshop, there are a few items that you will need to install or bring with you.
- a GitHub account
- Visual Studio Code IDE installed on your laptop


## :bomb: Exercise 1: Script injection in the run command

The [Check issue title workflow](.github/workflows/check-issue-title.yml) simply checks if the title of the workflow begins with octocat. If so, the workflow succeeds. If not, the workflow fails.  

This workflow is vulnerable to script injection. Let's find out why.  
[Exercise 1](./exercises/exercise-1.md)  

## :bomb: Exercise 2 - Script injection in github-script action

The [Check issue comment](.github/workflows/check-issue-comment.yml) workflow simply checks if the issue comment begins with `octocat`. If so, the workflow succeeds. If not, the workflow fails.  

This workflow is vulnerable to script injection. Let's find out why.  
[Exercise 2](./exercises/exercise-2.md)  

## :lock: Exercise 3 - Mitigate script injection in the run command

Let's see how we can mitigate script injection vulnerability in the run command.  
[Exercise 3](./exercises/exercise-3.md)  

## :lock: Exercise 4 - Migrate script injection in github-script action

Let's see how we can mitigate script injection vulnerability in github-script action.  
[Exercise 4](./exercises/exercise-4.md)  

## :mag: Exercise 5 - Mitigate using CodeQL Action Workflow

Let's create an actions workflow to scan our workflow files using CodeQL.  
[Exercise 5](./exercises/exercise-5.md)  

## :european_castle: Exercise 6 - Enhance the detection of vulnerabilities using third party queries

Now let's look at another way we can use CodeQL to secure our GitHub actions workflows.  
[Exercise 6](./exercises/exercise-6.md)  

## :books: Resources
- [Understanding the risk of script injections](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections)
- [Security hardening for GitHub Actions](https://docs.github.com/en/enterprise-cloud@latest/actions/security-guides/security-hardening-for-github-actions)  
- [CodeQL queries](https://github.com/advanced-security/codeql-queries/)  
