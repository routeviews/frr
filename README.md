# Route Views Collector 

This Route Views collector implementation is a fork of [FRRouting](https://github.com/frrouting/frr).

> This 'collector implementation' is really just a BGP router.
> However, we have a *special requirement*\* that requires us to maintain this patched version of FRR.
>
> \*: The *special requirement* is to provide 'VTYSH (CLI) access via telnet to the public.'

## Git Workflow

We use a GitHub Action Workflow to build new FRR packages for Route Views.

1. Checkout a new branch from the relevant upstream git tag, ex. [tag "frr-8.3.1"](https://github.com/FRRouting/frr/releases/tag/frr-8.3.1)
```
git checkout -b rv-frr-8.3.1 upstream/frr-8.3.1
```
2. Cherry pick [Route Views customization for FRR](#rv-patch-12-tags) (enabling running vtysh on the 'privileged' telnet port).
```
git cherry-pick rv-patch-1
git cherry-pick rv-patch-2
```
3. Also, cherry pick the [RV-CICD branch](#rv-cicd-branch) into the new branch (enabling GitHub Actions to run Earthly builds).
```
git merge rv-CICD
```
4. Remove any superfluous github workflows -- we only need ".github/workflows/build.yml" 
   1. Skipping this step can result in wasted 'GitHub Actions minutes!'
5. Update ".github/workflows/build.yml" to have only your new branch's name in the list of 'on.push' triggers:
```diff
name: Build FRR

on:
  push:
    branches:
-     - rv-frr-8.0.1
+     - rv-frr-8.3.1
```
5. Push the new branch up to GitHub. This will trigger a build!

At this point, load up the [Build FRR Workflow in GitHub](https://github.com/routeviews/frr/actions/workflows/build.yml) and monitor your build!

> **⚠ NOTICE:** We have an aggressive artifact time-to-live -- the built artifacts are only available for 5 days!
> Be sure to download them when the build completes!
>
> TODO: Consider using GitHub Packages to store these build artifacts! (If we get GitHub Enterprise)

> *"OH NO! The build failed!" ❌*
>
> **Not to worry!**
> You can review the build failure in GitHub, and make fixes in your branch.
> When you push up your fixes, GitHub will retry the build automatically! 🤞

### `rv-patch-[1:2]` tags

These commits contain changes that Route Views needs to enable serving up our core 'telnet to all our collectors' service.
In short, cherry picking these commits will "allow vty on privileged port"

### `rv-CICD` branch

The `rv-CICD` branch is a one-off orphan branch that holds our CICD solution for this repository.

> This separation of concerns allows our `master` branch to be kept in sync with the upstream FRR repository VERY easily.

### `master` branch

The `master` branch is simply kept in sync with the upstream as needed.

> No real development happens off of the master branch, so it is really okay for us to be lazy -- we can sync the fork at the last possible moment and it will be fine since we never push any changes to `master` branch ourselves.
