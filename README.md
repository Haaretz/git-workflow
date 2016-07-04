# git-workflow
A git workflow proposal for the Haaretz dev team

This proposal discusses two issues:  
- **[Branching strategy](#branching-strategy):** The branch structure and stages of a project
- **[Branching workflow](#branching-workflow):** The actual workflow of creating and working on feature branches.

For a short, printable cheatsheet of useful commands, see [here](#cheatsheet).

## Branching Strategy
The general branching strategy should follow the one outlined by the 
[Git Flow](http://nvie.com/posts/a-successful-git-branching-model/) model. 

Once we have test coverage good enough to move to a continuous integration model, 
we can consider adopting [GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html)
(see [this](https://guides.github.com/introduction/flow/) as well)

At its core, Git Flow is based on two main branches, which are always maintained and never go stale, and a multitude of temporary branches, used for specific, time-limited tasks, which will cease to exist once merged into one of the two main branches.

### Permanent Branches
The two permanent branches, `master` and `dev`, are used to aggregate changes developed in the temporary branches.

 - **master**: Stable, deployable code. `HEAD` is always production ready
    and each commit should represent a release.
 - **dev**: Represent latest _completed_ development changes for the upcoming release.
 
No code should ever be committed directly into one of the permanent branches, and the only commits 
in them should be merge commits.
 
### Temporary Branches
Three types of branches are used in the Git Flow strategy:
- [Feature branches](#feature-branches)
- [Release branches](#release-branches)
- [Hotfix branches](#hotfix-branches)

Each branch type may only be used for specific tasks, and are bound to strict rules as to which branches they can 
originate from, and which branches they can be merged into.

#### Feature Branches
**Branches from:** `dev`  
**Merged into:** `dev`

Used for developing a distinct piece of functionality. Feature branches exist only as long as the feature is in 
development, but will eventually be merged back into `dev` when ready (for inclusion in the upcoming release), 
or discarded entierly.

Feature branches are typically private, with only a single developer working on them.

For manageability and tracking, all code in the feature branch should be strictly restricted to the branch's 
topic and not exceed from it. For example, say a developer, while working on a feature branch, notices a small bug in the code base, that is unrelated to the feature. Fixing that code, despite requiring no more than changing a few characters, should _not_ happen within the feature branch, but rather in another, independent one. 

While this might seem extreme at times, it proves useful for several reasons:
1. Our feature branch might not end up being included in the next release. The bug fix should.
2. Our brunch may eventually end up being rejected and unused.
3. Compartmentalization allows us to better reason about pieces of code and its management.

Feature branches should only be merged into `dev` when they are ready to be included in the next release. Otherwise merging should wait until a new release cycle is started.

Merges to dev should only be done using the non fast forward strategy:
```sh
git co dev
git mnff <feature-branch>
```

Once merged into `dev` the feature branch is ready to be discarded:
```sh
git db <feature-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your branch both 
locally and _and from `origin`_. Only use it when after having finished merging your branch into
its base permanent branch.

#### Release Branches
**Branches from:** `dev`  
**Merged into:** `dev` and `master`

Release branches are used for preparing code for the next release. They are created when all features that are targeted for the release are already merged in to `dev`. All features targeted at future releases must not be merged into `dev` at this time, and should wait until after the release branch is branched off.

Release branches should be used for the QA process, bug-fixes of non-production code and preparing release meta-data. In order to minimize future merge conflicts, changes in a release branch should be continuously merged into `dev` immediately after being committed to the release branch.

Once complete, a release branch should be merged into `master`, where a tag should be created:
```sh
git co master
git mnff <release-branch>
git tag -a <release-number>
```

To keep the changes made in the release branch, which have not yet been merged back into `dev`, we need to merge 
those back into `dev` as well: 
```sh
git co dev
git mnff <release-branch>
```

This step may often lead to merge conflict, which should be resolved and committed into `dev`. The release branch may 
now be discarded:
```sh
git db <release-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your branch both 
locally and _and from `origin`_. Only use it when after having finished merging your branch into
its base permanent branch.


#### Hotfix Branches
**Branches from:** `master`  
**Merged into:** `master`, `dev` and release branch, if one exists

Hotfix branches are used to quickly patch code that has already been merged into `master`. This is the only branch that should fork directly off of master. As soon as the fix is complete, it should be merged into both master and develop (or the current release branch), and master should be tagged with an updated version number.

Once complete, a hotfix branch should be merged into `master`, where a tag should be created:
```sh
git co master
git mnff <hotfix-branch>
git tag -a <release-number>
```

and then into `dev` (or a release branch if one currently exists):
```sh
git co dev
git mnff <hotfix-branch>
```

The hotfix branch may now be discarded:
```sh
git db <hotfix-branch>
```

**Causion:** the `db` alias stands for `delete baranch`, and it will delete your branch both 
locally and _and from `origin`_. Only use it when after having finished merging your branch into
its base permanent branch.


## Branching Workflow
The commands outlined in this workflow assume you have the Haaretz 
[`.gitconfig`](https://github.com/Haaretz/htz-dotfiles/blob/master/.gitconfig) 
installed, and some of them are aliases, which will not be available with a 
vanilla installation of Git. It is generally a good idea to familiarize 
yourself with the config file, as it offers some handy aliases and shortcuts.

If it is not installed on your machine, please install it before doing anything else.
See instructions [here](https://github.com/Haaretz/htz-dotfiles).

### Working with Feature Branches
Before starting to work on your feature, checkout `dev` so it can be synced with `origin`:
```sh
git co dev
git nb <branch-name>
```

Your new branch is now checked out and ready to have code committed to it. Once you are done with the feature, 
go back to `dev`, make sure again that it is up to date, and return to your feature branch:
```sh
git updev
```

To maintain a clean, manageable tree, rebase your code onto `dev`:
```sh
# When feature branch is checked out
git rebase dev
```

This step may introduce conflicts, which will leave you in a `detached-head` state, mid-rebase.

Resolve the conflict(s) in your code editor of choice and then stage the changes (`git add`, `git rm` or `git reset HEAD <file>`).
See [here](http://gitforteams.com/resources/rebasing.html) for a greate tutorial on resolving mid-rebase merge conflicts.

Once all conflicts are resolved, run `git rebase --continue`.

If for any reason you would like to cancel the `rebase` process, run `git rebase --abort`. Whatever you do, do not run `git rebase --skip`, unless you know very well what you are doing.

Your code is now ready to be merged back into `dev`. Push it to GitHub and open a pull request against dev.

### Working with Hotfix Branches
Before starting to work on a hotfix, checkout `master` so it can be synced with `origin`, and branch off of that:
```sh
git co master
git nb <branch-name>
```

Your new branch is now checked out and ready to have code committed to it. Once you are done with the fix, 
go back to `master`, make sure again that it is up to date, and return to your hotfix branch:
```sh
git upmaster
```

To maintain a clean, manageable tree, rebase your code onto `master`:
```sh
# When hotfix branch is checked out
git rebase master
```

This step may introduce conflicts, which will leave you in a `detached-head` state, mid-rebase. See above on how to resolve that.

Once ready, hotfixes should be merged back to both `master`, `dev`. If there is an active release branch at the time the hotfix should also be merged into it.

## Cheatsheet

Some useful commands

| Command | Notes | Used for |
| --- | --- | --- |
| `git co` | `git checkout` | |
| `git mnff <branch>` | Merges `<branch>` into the branch you are currently without fast-forward. | Merging temporary branches back into permanent branches, while keeping a tidy tree structure. |
| `git db <branch>` | Deletes `<branch>` from both the local repo and from origin. | Discarding stale temporary branches _after_ they have been merged into the primary branch(es). |
| `git nb <branch>` | Syncs the branch your are currently in with origin, and creates a new `<branch>` branch. | Creating new `feature`, `release` or `hotfix` branches. |
| `git updev` | Checks out `dev`, syncs it with origin and checks out the branch you were originally on. | Preparing for rebasing a `feature` or `release` branch onto `dev`, just before it is ready to be merged back. |
| `git upmaster` | Checks out `master`, syncs it with origin and checks out the branch you were originally on. | Preparing for rebasing a `hotfix` branch onto `master`, just before it is ready to be merged back. |


### Workflow outlines:

#### Working on feature branches
Check out `dev`: `git co dev` --> <br />
&nbsp;&nbsp;Create a new branch: `git nb <branch>` --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;Work on feature --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Prepare branch for being merged back into `dev`: `git updev`, `git rebase dev` --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ solve conflicts ] --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Push to origin and open a pull request against `dev` --> Once your branch is merged, delete it: `git db <branch>`.

#### Working on release branches
Check out `dev`: `git co dev` --> <br />
&nbsp;&nbsp;Create a new branch: `git nb <branch>` --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;Prepare release --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Push to origin and open a pull request against `dev` and another pull request against `master` --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Once your branch is merged, into both `dev` and `master`, delete it: `git db <branch>`.

#### Working on hotfix branches
Check out `master`: `git co master` --> 
&nbsp;&nbsp;Create a new branch: `git nb <branch>` --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;Work on hotfix --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Push to origin and open a pull request against against `master` and another pull request against `dev` or against a release branch, if one exists --> <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Once your branch is merged, into both `master` and `dev` (or a release branch, if one exists), delete it: `git db <branch>`.
