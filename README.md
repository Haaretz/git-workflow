# git-workflow
A git workflow proposal for the Haaretz dev team

For a short, printable cheatsheet of useful commands, see [here](#cheatsheet).
For a short outline of commands, see [here](#workflow-outlines).

_note: The examples used throughout this document assume you have the Haaretz 
[`.gitconfig` file](https://github.com/Haaretz/htz-dotfiles/blob/master/.gitconfig) 
installed. Many of the command-line examples use aliases, which will not be 
available with a vanilla installation of Git. It is generally a good idea to 
familiarize yourself with the config file, as it offers some handy aliases and 
shortcuts.

If it is not installed on your machine, please install it before doing anything else.
See instructions [here](https://github.com/Haaretz/htz-dotfiles)._

## Branching Strategy
The general branching strategy outlined in this document, is an extension of
the [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/) model, with
the two most notable changes are its being based on a single permanent branch (`master`) 
instead of two, and releases being tag-based instead of branch-based. 

Once we have test coverage good enough to move to a continuous integration model, 
we can consider adopting [GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html)
(see [this](https://guides.github.com/introduction/flow/) as well)

## Branches
At its core, our workflow is based on a single permanent `master` branch, 
that is always maintained and never go stale, and a multitude of temporary 
branches, used for specific, time-limited tasks (`hotfix`, `feature`, `release`). 

### Permanent Branch
`master` is used to aggregate changes developed in the temporary branches. No code 
should ever be directly committed into `master`. The only commits it should be 
merge commits.

### Temporary Branch
There are three primary types of temporary branches:
  - [Feature branches](#feature-branches)
  - [Release branches](#release-branches)
  - [Hotfix branches](#hotfix-branches)

Temporary brnaches' sole uses are code isolation and backup. **They 
  should always be deleted once the changes in them have been merged into `master`**.

Each temporary branch type may only be used for specific tasks and should be 
kept isolated from each other.

#### Feature Branches
Each feature branch is used for developing a distinct piece of functionality. 
Feature branches exist only as long as the feature is in development, and will 
eventually be merged back into `master` when ready for inclusion in the upcoming 
release, or discarded entirely if rejected.

Though not mandatory, feature branches are typically private, with only a single 
developer working on them.

To create a new feature branch: 
```sh
git feature <branch-name>
```

For manageability and tracking, all code in the feature branch should be strictly 
restricted to the branch's topic and not exceed from it. For example, say a developer, 
while working on a feature branch, notices a small bug in the code base, that 
is unrelated to the feature. Fixing that code, despite requiring no more than 
changing a few characters, should _not_ happen within the feature branch, 
but rather in another, independent one. 

While this might seem extreme and annoying at times, it proves useful for several reasons:
1. Our feature branch might not end up being included in the next release. The bug fix should.
2. Our brunch may eventually end up being rejected and unused, which in turn mean 
   the bugfix never was also rejected.
3. Compartmentalization allows us to better reason about pieces of code and its management.

Feature branches should only be merged into `master` when they are ready to be 
included in the next release. Otherwise merging should wait until a new release 
cycle is started.

Before being merged back into `master`, feature branches should be rebased onto 
`master`, so that our history remains linear and readable. Be aware though, that 
this _w i l l_ rewrite history. **Rebase should only be done in private branches,
which were not shared with other developers, and should only right before you are 
ready to merge changes into `master`.**.

After bringing the feature branch up to date with `master`, merge it back using  
the no-fast-forward strategy:

```sh
#  From feature branch:
#######################
git upmaster                          ## Update master branch with origin
git rebase master                     ## Carful with this one
git push -u origin <feature-branch>   ## Only merge after branch is approved
git co master                         
git mb <feature-branch>               ## mb stands for 'merge branch'
```

Once merged into `master` the feature branch should be discarded:
```sh
git db <feature-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your 
branch both locally and _and from `origin`_. Only use it after having finished 
merging your branch into `master`.


#### Release Branches
Release branches are used for preparing code for the next release. They are 
created when all features that are targeted for the release are already merged 
into `master`. All features targeted at future releases must not yet be merged 
into `master` at this time, and should wait until after the release branch is 
branched off.

Once you branch off to a release branch, features for the next release can 
continue to be safely merged into `master` as usual.

To create a new feature branch: 
```sh
git feature <branch-name>
```

Release branches should be used for the QA process, bug-fixes of non-production 
code and preparing release meta-data. In order to minimize future merge conflicts, 
changes in a release branch should be continuously merged into `master` 
immediately after being committed to the release branch.

Once complete, the tip of the branch should be tagged with a version number in 
order to cut the release. It is the tag that serves as the deployment point, not 
the branch.

Once tagged, the release branch should be merged into `master` and deleted. 
```sh
#  From the release brnach
##########################
git tag -a <release-number>
git co master
git mb <release-branch>
```

This step can often lead to merge conflict, which should be resolved and committed into `master`. 

The release branch may now be discarded:
```sh
git db <release-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your branch both 
locally and _and from `origin`_. Only use it when after having finished merging your branch into
its base permanent branch.


#### Hotfix Branches
Hotfix branches are used to quickly patch code that has already been deployed 
into production. Hotfix branches fork off the latest tag, not off a branch:

```sh
git hotfix <branch-name>  # An alias that checks out the the latest tag and 
                          # automatically creates a new branch off of it.
```

As soon as the hotfix is complete, the tip of the branch should be tagged with 
an updated version number in order to cut the new patch release. It should then 
be merged into `master`, as well as the active release branch, if one exists.

```sh
#  From hotfix brnach
##########################
git tag -a <release-number>
git co master
git mb <hotfix-branch>
```

The hotfix branch may now be discarded:
```sh
git db <hotfix-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your branch both 
locally and _and from `origin`_. Only use it when after having finished merging your branch into
its base permanent branch.


## Cheatsheet
Some useful commands

| Command | Notes | Used for |
| --- | --- | --- |
| `git co` | `git checkout` | |
| `git mb <branch>` | Merges `<branch>` into the branch you are currently using the no-fast-forward strategy. | Merging temporary branches back into `master`, while keeping a tidy tree structure. |
| `git db <branch>` | Deletes `<branch>` from both the local repo and from origin. | Discarding stale temporary branches _after_ they have been merged into the primary branch(es). |
| `git feature <branch>` | Syncs `master` with `origin` and checks out a new branch from it | Creating new `feature` branches. |
| `git release <branch>` | Syncs `master` with `origin` and checks out a new branch from it | Creating new `release` branches. |
| `git hofix <branch>` | Syncs tags with `origin` and checks out a new branch from from the latest tag in the refs | Creating new `hotfix` branches. |
| `git upmaster` | Checks out `master`, syncs it with origin and checks out the branch you were originally on. | Preparing for rebasing a branch onto `master`, just before it is ready to be merged back. |


## Workflow outlines:
### Working on feature branches
  - Create a new feature-branch: `git feature <branch-name>`
  - Work on feature
  - Prepare branch for being merged back into `master`: `git upmaster && git rebase master`
  - [ resolve conflicts ]
  - Push to origin and open a pull request against `master`
  - Once your branch is merged, delete it: `git db <branch>`

### Working on release branches
  - Create a new release-branch: `git release <branch-name>`
  - Prepare release
  - tag release when it is ready: `git tag -a <release-version>`
  - Push to origin and open a pull request against `masterr`
  - Once your branch is merged, delete it: `git db <branch>`.

### Working on hotfix branches
  - Create a new branch off of the latest deployed release: `git hotfix <branch-name>`
  - Work on hotfix
  - tag hotfix when it is ready, to create a new release: `git tag -a <release-version>`
  - Push to origin and open a pull request against against `master` and another 
    pull request against the active release branch, if one exists
  - Once your branch is merged, delete it: `git db <branch>`.
