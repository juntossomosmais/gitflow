# Gitflow: Backend

The main gitflow is very easy to follow, but includes some variations that might happen on our day-to-day work and are also explained here.

**Content**

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Main gitflow](#main-gitflow)
    - [Summary](#main-gitflow-tldr)
- [Variation: flow with extra sync (dev -> release -> prd)](#variation-flow-with-extra-sync-dev---release---prd)
- [Variation: flow with release reuse](#variation-flow-with-release-reuse)
- [Variation: flow with two features on dev](#variation-flow-with-two-features-on-dev)
- [Variation: hotfix](#variation-hotfix)
- [Variation: broken dev](#variation-broken-dev)
    - [Fix it](#fix-it)
    - [Remove changes](#remove-changes)

<!-- /TOC -->

---

## Main gitflow

![Main flow](../../assets/backend/gif/main-flow.gif)

The main flow is easy to follow and the one we should strive to achieve. But, first things first, you should pair your local `develop` branch with what is on remote.

```sh
git checkout develop
git pull
```

Then, it's time to create your feature branch:

```sh
git checkout -b feature/xpto # or git branch feature/xpto && git checkout feature/xpto
```

If your local development workspace doesn't have the `feature/xpto` branch, the command above will create it and switch your workspace to the newly created branch!

Now, it's time to code! After you made your changes and think you have enough code to constitute a commit (a package with changes), create it with:

```sh
git add changed-file # if you want to add all changes made to all files into one commit: git add .
git commit -m "nice commit message" -m "More description" \
           -m "How many paragraphs can I stack?" -m "I don't know" # or `git commit` to bring your git editor (usually `vim`)
```

Then, when you feel confortable, just push your changes to remote!

```sh
git push -u origin feature/xpto
```

This will push any commit you did on your feature branch to remote and set the remote branch as a source reference to your local branch, so that any `git pull` and `git push` you do later, when working in this branch, is done referencing the remote one.

Many `git add`, `git commit` and `git push` later, it's time to bring your new feature to the remote `develop` branch and update the development environment with your changes. But, before that, we want to evaluate what you did, checking if the changes address what the task asked, if there are enough tests, if the code can introduce any obscure bug, etc. And, for this, we run through a process called **Code Review**. It's an important step to prevent any undesirable change and to leverage the knowledge across the team.

So, now, you need to create a merge request or a Pull Request (PR)! For that, use whatever tool you want! You could use the GitHub UI ([step guide](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request)) or the GitHub CLI tool ([manual](https://cli.github.com/manual/gh_pr)). After its creation, call for another DEV to review your changes. After some discussion and possible changes and only after you both agree that the feature can go on, merge your branch to `develop`!

With everything smooth on our development environment, you can delete your feature branch and create a new release branch!

```sh
git push -d origin feature/xpto # delete the feature branch from origin (remote)
git checkout develop && git pull
git checkout -b release/xpto
git push -u origin release/xpto
```

Now, with the QA, or homolog, environment updated, call for a QA analyst to test your brand new feature! After her approval and with the product owner (PO) consent, create a PR targeting the main production branch (usually `master`) and call for another DEV approval.

When it's approved, merge the PR and you're almost done! Take a time to monitor how your change affected the production environment and create a changelog to communicate everyone!

And, now we're done! Congrats!!!! :tada: :tada: :tada:

<a name="main-gitflow-tldr"></a>
### Summary

1. `git checkout develop && git pull`
2. `git checkout -b feature/xpto && git push -u origin feature/xpto`
3. Repeat `git add && git commit`, eventually `git push`, until the feature is done
5. Create a Pull Request (PR) targeting the `develop` branch and call for any DEV to review it
6. Do the requested changes, if necessary, until approval
7. Merge into `develop` and delete your feature branch
8. `git checkout develop && git pull && git checkout -b release/xpto && git push -u origin release/xpto`
9. QA Test
10. After QA and PO approval, create a PR targeting the `master`, or `main`, branch
11. After the PR approval, merge it and you're almost done!
12. Now, it's time to keep an eye if everything is fine with your feature in production and to create a changelog so everyone knows what you did!

## Variation: flow with extra sync (dev -> release -> prd)

A very common situation that may arise once a new release branch is created is something that definitely has happened to every developer out there (unless, well, he wasn't a human at all): a bug.

Even when the code seems to be working fine and all the tests are passing, things can still go wrong and hopefully you will have a QA analyst to help you identify the issue before the code actually reaches production.

In these situations, the gitflow allows developers to make commits **directly to the release branch** in order to fix the release but comes at the cost of creating bugfix code outside the develop branch. This is bad as it drives the `develop` branch farther from the code that is actually going to production and this violates one of the golden rules of this gitflow: to keep `develop` and `master` as close as possible as **every new released feature will come from the development branch**. Hence, if commiting to the `release/*` branch is a must, then do it but **do not ever forget** create two PRs when the `release/*` has been validated:

- One PR to `master` (the actual code going to production)
- One PR back to `develop` (yes, this is required to keep master, dev and future releases with the required bugfix code)

So, here's the flow for this scenario:

![Alt text](../../assets/backend/gif/variation_extra_sync_flow.gif)

And a final static image to illustrate the whole flow:

![Alt text](../../assets/backend/img/variation_extra_sync_flow.png)

## Variation: flow with release reuse

This variation also may be very common and depends a lot on how many developers are working with the same code base. This case usually happens when two or more developers finish writing their code on their respective feature branches, for example: `feature/feat-A` and `feature/feat-B`, and they both send the finished code to the `develop` branch. In this case, the `develop` branch will have **more than one feature** that should go to production at the same time.

This scenario can happen in two ways:

1. The two features are on the `develop` branch, but no `release/*` branch has been created yet;
2. One the features, for example `feature/feat-A`, has already been used to create a release `release/feat-A` whereas the other feature, `feature/feat-B` is "waiting" on the `develop` branch

In both cases, one thing is totally required: **team communication**. When this situation happens, the **team** must decide if the two features can be shipped together to production or not.

Let's discuss these two cases:

### IF the two features cannot go together to production

If the team agrees that the two features cannot go together to production, then there's nothing much to be done: `feature/feat-B` will have to wait until `release/feat-A` is merged to master and then a new release can be created `release/feat-B`.

Going back to the two ways that this can happen:

1. The features can't go together to production, hence `feature/feat-B` must be removed from the `develop` branch so that `feature/feat-A` can be used to create its `release/feat-A`. `feature/feat-B` will have to wait.

2. In this case, as `release/feat-A` already exists, `feature/feat-B` can wait on `develop` until the other feature reaches production. After that happens, the other release `release/feat-B` can be created.

`PS`: Bear in mind that this scenario can create **bottlenecks** as one feature will have to wait on the other one before being tested and release to production.

### IF the two features CAN go together to production

After the team has agreed to release the two features together, then:

1. In this case, a single release can be created which will contain the two features together: `release/feat-A-and-B`
2. Here, as a release branched already exists, then this release can be **reused** to contain the two features. The most common way to handle this is by rebasing the `release` branch onto the top of the `develop` branch (so that the two features will be on the same `release` branch). This is called `release reuse`.

Here's the flow for the `release reuse`:

![Alt text](../../assets/backend/gif/variation_release_reuse.gif)

And a final static image to illustrate the whole flow:

![Alt text](../../assets/backend/img/variation_release_reuse.png)

## Variation: flow with two features on dev

## Variation: hotfix

![Hotfix flow](../../assets/backend/gif/hotfix.gif)

Although this flow should usually be avoided, it might happen. So, when a wild serious bug appears in production and you need to fix it as soon as possible, it's time to create a hotfix!

The first step is to create your hotfix branch from the main branch. Supposing it's `master`, do the following:

```sh
git checkout master && git pull # remember to always be updated!
git checkout -b hotfix/bug
```

After fixing the bug and adding some tests for it, it's time to create the PR targeting `master`. Then, after the changes integrate on production and you guarantee that it fixes the bug, you **must** create a PR for `develop` so that the fix is also integrated there.

Finally, don't forget to release the fix to the QA environment (creating a `release/*` branch from `develop`). Although it can be done later, it's always important to have all environments stable and up to date!

## Variation: broken dev

It's important to say that we should not break our development environment at all costs. It's shared between all devs and we need to ensure that it's stable enough so that everyone can work with and use it for their tasks. Saying that, we know that it can happen. We can break DEV, we can release into it a half-baked feature. And, it's totally fine, because:

> To err is human

Consider it a lesson to learn and if that happen, we have two alternatives: fix it or remove the changes from `develop`.

### Fix it

![Fix develop](../../assets/backend/gif/fast-fix-dev.gif)

If possible, we should always try to fix `develop` before any other change, because the flow is simpler. As any other feature, we start a new branch from `develop` (or merge the `develop` changes into your broken feature branch) and follow the normal development flow!

### Remove changes

![Remove commits from develop](../../assets/backend/gif/remove-dev.gif)

But, if we know that the fix will take some time, it's important to revert the introduced changes. For that we have two alternatives:

- Reset the git codeline, as if the feature wasn't introduced to branch `develop`

Suppose that you're at the top (`HEAD`) of your branch and the commit sitting there is a merge commit that came from your PR. If you want to remove it:

```sh
git reset --soft HEAD^ # use `--hard` instead of `--soft` if you don't want the changes
git push --force # be very careful!
```

This will remove the commits associated with the last merged PR and rewrite the git codeline.

Notice here that we used `git push --force`. It's not something we should do with frequency because it affects everyone that is sharing the same repository and if someone just did a pull with your changes before the forced push, rewriting the timeline will just blew up her local repository. Thus, it's preferable to avoid it. That leads to our second approach:

- Revert the commits, creating a commit that revert them

Again, suppose that you're at the `HEAD` of your branch and the merge commit is the one sitting there. Then:

```sh
git revert -m 1 HEAD # or the SHA commit you want to revert
```

The command above will create the commit you want, effectively reverting all the changes introduced with the merged PR. We can also do it using the GitHub UI ([step guide](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/reverting-a-pull-request)) and it's quite easy!

The thing we should be aware is that if we want our changes back we need to revert the revert. So, again, do a revert:

```sh
git revert HEAD # or the SHA commit
```

And continue your work!