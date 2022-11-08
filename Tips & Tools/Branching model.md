## [[git-model.png |Diagram]]
- `origin` is the single source of truth, *the one* repo that truly matters.

- `origin/master` is the main branch where the source code of `HEAD` **always** reflects a 
**_production-ready_** state. Every commit on `master` is a new release _by definition_ (i.e. must be version-tagged for easy future reference to this historical version).

- `origin/develop` is the main branch where the source code of `HEAD` **always** reflects a state with the latest delivered development changes for the *next* release. When the source code in the `develop`branch reaches a *stable* point and is *ready to be released*, all of the changes *should be merged* back into `master` and then tagged with a release number.

- Other supporting branches to aid parallel development between team members are `feature`, `hotfix`, `release`. There names are categorized by how we _use_ them. Unlike the main branches, these branches always have a limited life time, since they will be removed eventually.

- `feature` branches (or sometimes called topic branches) are used to develop new features for the upcoming or a distant future release. They may branch off from `develop` and exists as long as the feature is in development, but will eventually be merged back into `develop` (to definitely add the new feature to the upcoming release) or discarded (in case of a disappointing experiment). 
  Branch naming convention could be anything except `master`, `develop`, `release-*`, or `hotfix-*`. They typically exist in developer repos only, not in `origin`. Practical example:
```bash
# Create a feature branch from `develop`:
$ git checkout -b myfeature develop
-----------------------------------
# Merging a `feature` into `develop`: 
$ git checkout develop    # Switched to branch `develop`

$ git merge --no-ff myfeature # Updating ea1b82..05e957 (Summary of changes)
# Optionally:
$ git branch -d myfeature  # Delete branch `myfeature` (was 05e955).

$ git push origin develop
```

- `hotfix` branches are created from the `master` branch. Branch naming convention: `hotfix-*`. For example, say version `6.9` is the current production release running live and causing troubles due to a severe bug, but changes on `develop` are yet unstable:
```bash
# Switched to a new branch "hotfix-6.9"
$ git checkout -b hotfix-6.9 master

# Explicitly mention the version being fixed (i.e. version bumped to 6.9)
$ ./bump-version.sh 6.9

$ git commit -a -m "Bumped version number to 6.9"
# [hotfix-6.9 41e61bb] Bumped version number to 6.9
# 1 files changed, 1 insertions(+), 1 deletions(-)
----------------------------
# After fixing the bug:
$ git commit -m "Fixed critical production problem"
# [hotfix-6.9 abbe5d6] Fixed critical production problem
# 5 files changed, 32 insertions(+), 17 deletions(-)

# When finished, the bugfix needs to be merged back into `master`, but also needs to be merged back into `develop`, in order to safeguard that the bugfix is included in the next release as well:
# Update `master` and tag the release
$ git checkout master

$ git merge --no-ff hotfix-6.9

$ git tag -a v6.9 -m "fixed some bugs in some features"

# Include the bugfix in `develop`:
$ git checkout develop

$ git merge --no-ff hotfix-6.9

$ git branch -d hotfix-6.9      # Optional
```
When a `release` branch currently exists, the `hotfix` changes need to be merged into that release branch, instead of `develop`.

- `release` branches support preparation of a new production release. They're branched off from `develop` , must merge back into `develop` and `master`. They allow for minor bug fixes, trivial tweaks of a feature and preparing meta-data for a release (version number, build dates, etc.). 
By doing all of this work on a `release` branch, the `develop` branch is cleared to receive features for the next big release. A tag is assigned at the creation point of a `release` branch. During its lifetime for a version, bug fixes may be applied in it and then merged back into `develop` later (rather than directly on the `develop` branch). Adding large new features here is strictly prohibited. Branch naming convention: `release-*`
```shell
$ git checkout -b release-6.9 develop

$ ./bump-version.sh 6.9

$ git commit -a -m "Bumped version number to 6.9"

$ git checkout master

$ git merge --no-ff release-6.9

$ git tag -a 6.9 -m "Offical release v6.9"

# To keep the changes made in the release branch, we need to merge those back into `develop`:
$ git checkout develop

# Most likely a merge conflict will occur arounf this step:
$ git merge --no-ff release-6.9

$ git branch -d release-1.2   # Optional
```