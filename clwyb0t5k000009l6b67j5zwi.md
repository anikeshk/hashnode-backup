---
title: "GitHub Actions"
seoTitle: "GitHub Actions"
seoDescription: "Introduction to GitHub Actions with an example of running unit tests on commit pushes"
datePublished: Mon Jun 03 2024 01:40:44 GMT+0000 (Coordinated Universal Time)
cuid: clwyb0t5k000009l6b67j5zwi
slug: github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717375536563/2b253be2-26d4-4457-bc72-ffbd2737efde.png
tags: github, github-actions, github-actions-1

---

## Introduction

This articles introduces you to **GitHub Actions** and provides an example GitHub Action that runs unit tests on each GitHub commit push.

## Github Actions

**GitHub Actions** is an additional product offering from GitHub Actions to aid developers in Continuous Integrations in their projects. It allows developers to build **workflows** (like running tests or building an image) on certain **events** (like pushing code to GitHub). GitHub uses **runners** to execute these tasks, which is a virtual machine (usually based on Linux, but can be customized to other OSes).

Each workflow consists of a series of steps called **jobs**. These steps may include building your code, running tests, or re-using existing actions available on the GitHub Marketplace.

## Example

Let us take a look an example GitHub Action.

```bash
name: Run Jest Tests

on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run Jest tests
        run: npm test
```

1. The `on` part of the file tells GitHub Actions which events should trigger this workflow. In this case, it is commits on pushed to the `master` branch or when a pull request is merged to the `master` branch.
    
2. The `jobs` part of the file tells GitHub Actions what steps to execute. In this case, first Node.js v20 is setup on a Ubuntu Linux virtual machine. Then the dependencies are installed, and then the tests are run.
    

If you take a look at the Actions tab in a GitHub repository, you will be able to see the logs and status for each GitHub Action.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717378633059/d0a0555f-3b8d-4183-9420-6e5198f92daf.png align="center")

If you click any of the workflow runs, you will get a detailed list of steps executed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717378716956/b2a0de21-6a59-489f-b92e-c54f370e826d.png align="center")

In the above screenshot, we can see the run for "Add GitHub Action for tests #2" failed because some of the tests didn't pass.

On fixing the code and pushing to the main branch, the "Add missing author field in createBook #3" workflow run passed and has a green checkmark.

This example is from: [https://github.com/anikeshk/ci-books](https://github.com/anikeshk/ci-books)

You can find the GitHub action in the `.github/workflows` folder: [https://github.com/anikeshk/ci-books/blob/master/.github/workflows/jest-test.yml](https://github.com/anikeshk/ci-books/blob/master/.github/workflows/jest-test.yml)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This is one example GitHub Action workflow. There are many more workflows that lint code, build images, perform static analysis of your code, and more! And they can be configured to run on each commit, when pushed to a certain branch, or manually. Check the References section for more examples. </div>
</div>

 <div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">GitHub Actions can be setup for any language supported by GitHub. Java and JavaScript/TypeScript are the popular options.</div>
</div>

## References

[Official GitHub Documentation for GitHub Actions](https://docs.github.com/en/actions)

<iframe width="560" height="315" src="https://www.youtube.com/embed/eB0nUzAI7M8?si=0MnVD1AWnJSRQQQO"></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/yfBtjLxn_6k?si=8VQoCuG9vNsrFTtB"></iframe>