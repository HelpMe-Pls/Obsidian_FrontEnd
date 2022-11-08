[[Branching model]]
[[git-commands.png |Common commands]]


```bash
# Quick git cmd when working on a file:
git add <filename(s)>    # (stage file(s))
git add .                # (stage all changes in cwd)
git commit -m <Title> -m <description>
git commit -am <message> # (commit with staged change i.e. you can skip git add with this)
git push
```

### Merge, merge --squash, Rebase 
- **Merge:** use it if you want to have the all commits in chronological order from both branches and also have an extra "merge commit" in the tree at that merging moment.
- **Merge --squash**: use it if you just wanted to have the updates from the target branch but NOT their commits and also NOT wanting to have an extra commit at that moment, then later have a commit to represent your current work (not necessarily a merge commit).
- **Rebase**: use it ***BEFORE PUSH*** if you want to have a ***clean*** git tree (i.e. the end of branches not contacted with each other) and you're really sure that your commit messages look meaningful so that the `target` branch looks easy to read/understand.
Rebase `src` onto `target` means taking every commits from `src` and put it on top of `target` (`src` is still the current checkout, but the git tree looks like your `src` is branched of from the tip of `target` at the point of executing `rebase`, i.e `src` still has all the updates from `target`, it just looks different in the git tree as if your `src` is now growing from the tip of `target` with ALL the commits from both branches )
So that's why if there're conflicts, you'll have to resolve the conflicts *one-by-one* (coz the `target` is now the "base" and it just had *more commits* from your `src` ), not all at once like the `merge`

--- 
How to work with git in TFS:
1. Fork project
2. Create your `feature` branch (branch name depends on sprint/feature/...)
  2.1. Create another remote for origin (name: origin, push & pull URL === clone URL)
  2.2. Bring 2 branches (remote origin/master and remote forked_repo/my_branch) to your local on gitKraken  
3. Dev on your branch + pull master (or checking out master, then merge MASTER INTO YOUR BRANCH if u wanna update the latest stuff from the remote)
4. Commit & push your code to YOUR remote branch  
5. Create PR whenever u want to merge that feature to origin/master (if that PR isn't merged then new pushes will be included in it)
  5.1. Merge PR can only be done by the branch owner, code reviewers will give opinion if you're allowed to merge or not.
6. If PR is approved, hit Complete then it'll auto merge YOUR BRANCH into the COMPANY'S remote MASTER


---
Questions:
- After merging a branch, is that branch gone and should i create a new branch to update a feature from that branch or should i create a new branch ? 
    > It depends on the purpose of the branch (sprint, hotfix, adding components,..), if u wanna continue developing that feature, go ahead keep working on that branch, then create new PR once your stuff is done. 
    > If u wanna create new feature or moved on to the next sprint then create new branch.

- I'm on my branch, someone updated remote master and i wanna have that piece of latest code in my branch also, what to do ? 
> 	Merge master into your branch (then push it if u want remote master to have your current stuff too)

- If i want to update a file in some random previous commits and NOT wanting it to merge with the current code flow, what should i do ?
    > Create a branch from that commit then do whatever u want on it

- Add new untracked files to .gitignore: 
  - Unwanted file created, it will appear in git staging area
  - Add that file's name to .gitignore then save OR right click on it from the staging area, there will be an option for "add to .gitignore"

- How to add to .gitignore previously tracked file(s) or folder(s) ?
    > cd desired/file/folder/path
    > git rm --cached <filename> 
 or
    > git rm -r --cached <foldername> 
    > Add the <filename> || <foldername> to .gitignore and Ctrl+S it
    > git commit -am "Remove ignored files" or enter commit message then Ctrl+Enter
    > push
  >> To undo that:
    > git rm -r --cached .
    > delete the <filename> || <foldername> from .gitignore and and Ctrl+S it
    > git commit -am "Recover ignored files" or enter commit message then Ctrl+Enter
    > push

  >> Files that are added to .gitignore will be excluded in the remote repo after any push command. In case you still want to remote to have that file/folder but not updating new changes from now, use these commands:
    > cd desired/file/path
    > git update-index --skip-worktree <filename> 
	> To undo that: git update-index --no-skip-worktree <filename> 
    </> For a folder with multiple files:
        > cd /its/parent/folder	
	> git ls-files -z public/ | xargs -0 git update-index --skip-worktree
	> To list all files that are skipped in that folder:
	    > cd to/its/path
	    > git ls-files -v . | grep ^S | cut -c3-
	> To undo that:
	    > cd to/its/parent
	    > git ls-files -z <foldername>/ | xargs -0 git update-index --no-skip-worktree
    </> For a folder with subfolders and files: 
	    > cd to/topmost/folder
	    > find . -maxdepth 1 -type d \( ! -name . \) -exec bash -c "cd '{}' && pwd && git ls-files -z ${pwd} | xargs -0 git update-index --skip-worktree" \
	 > To undo that:
	    > cd to/topmost's/parent
	    > find . -maxdepth 1 -type d \( ! -name . \) -exec bash -c "cd '{}' && pwd && git ls-files -z ${pwd} | xargs -0 git update-index --no-skip-worktree" \
  >> You can replace --skip-worktree with --assume-unchanged, following these scenarios:
    > --assume-unchanged assumes that a developer SHOULDN'T change a file. This flag is meant for improving performance for not-changing folders like SDKs. Listing skipped file with cmd: git ls-files -v|grep '^h'|cut -c3-
    > --skip-worktree is useful when you instruct git not to touch a specific file ever because developers SHOULD change it. For example, if the main repository upstream hosts some production-ready configuration files and you don’t want to accidentally commit changes to those files.

	    
	

 
