# git-sandbox
A place you can go and play with git without getting into (too much) trouble.

## Getting Started
The first thing to decide is whether to just `clone` the desired repository or 
`fork` and `clone` it.  If I know that I will be making changes, I prefer to work off
a fork, otherwise I just clone the repo.  Where this makes a difference is whether
you need to manage `remote` repos.

Forking a repo takes place from the github UI.  Go to the desire repo and use the `Fork` 
button in the upper right corner of the page.  You'll then be asked of where you'd like
to place a copy of the repo, with the default in your local (self) organization.  Once
forked (or not) the clone is the same:

`git clone https://github.com/<org>/<repo>.git`

The last parameter can be cut/paste from the github UI when in the appropriate 
organization.

## Setting up a Remote
If you forked the repo, you'll want to add a remote repo that corresponds to the 
repository from which your fork was created.  Typical naming conventions are:

`origin`: denotes the location of the default repository.  In forked configurations
this will be the destination repo of the fork operation.

`upstream`: denotes the location of the source (official) repository and really only 
comes into play in forked configurations.  This repository will be the target of any 
pull requests (PRs) you submit from your branch.

`xyz`: You might have other repositories (typically of teams or colleagues) in which
you may want to work.

- To see a list of the repos in use, use: `git remote -v`
- To add a remote to your current list, use: `git remote add <remote-name> <repo URL>`

For example:  If there was a git-sandbox repo in the apache organization one might use:

`git remote add upstream https:/github.com/apache/git-sandbox.git`

Once issued, `git remote -v` should display the repositories available for this working
area.

- To rename a remote, use: `git remote rename <old-name> <new-name>`
- To remove a remote, use: `git remote rm <name>`

## Synchronizing a working area from the master
To update your working area (and local repo) with changes applied to master, do the following:

- `git fetch upstream` # Makes changes from `upstream` available
- `git checkout master`  # Switch to the master branch
- `git rebase upstream/master`  # Apply changes from master to your local work area
- `git push origin master`  # Push local changes to github repo denoted by `origin`

If you also had a dev branch that you wanted to update with master...
- `git checkout dev-branch`  # Switch to the dev branch
- `git rebase master`  # no need to use `upstream` since master is already up to date

## Making changes
To make changes that are then (eventually) applied to the master source base, do the
following:

- Synchronize your work area using the steps described above and create a working branch:
- `git checkout -b dev-branch`
- Make and test necessary changes
- `git add .`  # Prepare all files for commit.  If only a subset of changed files are
to be committed then only `add` those.
- `git commit -m <comment-heading>`  # commits the files locally.  Use a brief statement 
that summarizes the changes.
- `git commit --amend`  # This command is used to bring up an editor (vi) that allows you
to add more verbose comments.  In both cases, try to limit the characters in each line
to 70.  This makes life easier to committer that need to process the changes.
- `git push origin dev-branch`  # This pushes the changes to the repo denoted by `origin`
and creates a `dev-branch` branch.

Once changes are in place, a pull request can be created.  This is accomplished via the
github UI using the "Create Pull Request" button.  If these changes are relative to an
Issue, the comments should include `Fixes #<Issue Number>` at the end.

## Deleting branches
Once changes have been submitted and merged into master, its best to remote the dev branch.
This must be done both locally and from the github UI.  For the latter, github is usually
pretty good about providing a button saying that its okay to delete the branch.  To clean
up the local branch use:
- `git branch -d dev-branch` # Sometimes, depending on the way the merge was done, it may
be necessary to require a forced deletion by replacing `-d` with `-D`.

## Squashing commits
Its best to submit a PR of one commit - although not required.  To squash a set of commits 
into one, use the following:

- `git log` # Presents a list of commits ordered by date - newest to oldest
- Identify the commit hash on which the to-be-squashed commits were applied and 
call that "original-commit-hash". I.e., the starting point.
- `git rebase -i <original-commit-hash>`  # This is an *interactive rebase* and will 
bring up an editor with the set of to-be-squashed commits  prefixed with 'pick'.  
Change 'pick' to 's' for all but the first one of those items (leaving one to act as the primary commit).
(Note: When merging PRs (below) `git rebase -i master` from the pr branch only gives commits unique 
to your branch.)
- Edit the header of the next entry to have the appropriate (encompassing) message.
- Push the changes (preferrably to a new branch), delete the original branch and recreate it or,
if merging another PR and working in the "pr branch", leave alone.
- To push to a new branch...
	- `git push origin <prev-branch>:<new-branch>`
- To delete and recreate the original branch...
	- `git push origin :<prev-branch>`   (to delete)
	- `git push origin <prev-branch>:<prev-branch>`  (to recreate - although haven't done this yet)
- Once pushed (or not in PR merge case) - review the commit and ensure the changes look correct.

## Merging changes from master onto dev branch
Here are steps to merge a commit from a dev branch to the master branch.  In summary, 
the steps first update the dev branch with the latest changes from master, resolving 
any conflicts that may occur, then cherry-picks the last commit from the newly updated
dev branch (which now has master as its basis) back onto master
and pushes those updates to applicable repos (upstream and origin).

##### Merging changes from master onto dev branch

- `git fetch upstream` or `git fetch --all` # Get the latest from upstream (et al)
- `git checkout master` # Change to local master branch
- `git rebase upstream/master` # Apply upstream changes to local master branch
- `git checkout dev-branch` # Change to local dev-branch
- `git rebase master` # Apply changes from local master to current branch
- Previous command encountered conflicts.  Use pyCharm VCS->Git->Resolve 
Conflicts to fix middle panel.  Once resolved, continue rebase.
(Note: If something goes wrong or a merge doesn't look correct,
issue the following... `git rebase --abort`)
    - git rebase --continue

*TEST CHANGES HERE...*

- `git push --force origin dev-branch` # Local dev branch updated, push to repo (fork in this case)
- `git commit --amend` # Fixup commit comments, add Closes #NNN or whatever.
- `git push --force origin dev-branch` # Push those updates back to repo

If multiple commits are to be merged, its best they be squashed into
a single commit...
- `git checkout dev-branch` Get onto the dev branch (probably there already)
- `git rebase -i master`  # Perform interactive rebase (see above), updating all entries except 
the first which becomes the 'squash commit'.  Change 'pick' to 's' for non-first entries.

##### Merging dev branch commit back to master (i.e., merge the PR) 
- `git log` # Grab the commit hash from the last (first) commit listed in the log 
- `git checkout master` # Switch to local master branch
- `git log` # Look at the log just to check that things look okay (should not see
anything regarding the changes to be merged) 
- `git cherry-pick <selected-hash>` # Cherry pick the commit (using the hash select in first git log step)
- `git log` # Look at the log to make sure cherry-picked commit is present
- `git push upstream master` # Push merged changes to upstream master branch
- `git push origin master` # Push merged changes to forked repo master branch


## Merging another's PR
Handy merging aliases - add to [Alias] section of ~/.gitconfig:

```text
[alias]
amenddate='git commit  --amend --date=now'
pro = "!f() { git fetch ${2:-origin} pull/$1/head:pr-$1 && git checkout pr-$1; }; f"
pru = "!f() { git fetch ${2:-upstream} pull/$1/head:pr-$1 && git checkout pr-$1; }; f"
```
- Create clean clone in separate directory
 	- `cd merging`
 	- `git clone <repo>`
 	- `cd repo-dir`
 	
- Since clean repo uses origin as the truth, use 'pro' alias (PR origin) using the PR number (303)
 	- `git pro 303`

- Now on branch pr-303, check commit log
 	- `git log`

- If this is from an issue, ensure 'Fixes #<Issue>' is present.  Add 'Closes #<PR>'. Amend the commit.
 	- `git commit --amend`

- Switch back to master
 	- `git checkout master`

- Check log, if there are commits that weren't in pr-303 besides the PR commit, then need to rebase
master to pr branch
 	- `git log`

- Additional commits present, go back to pr branch
    - `git checkout pr-303`

- Rebase master to the pr branch
    - `git rebase master`

- Check log to see that only pr commit is different
 	- `git log`

- Go back to master branch
 	- `git checkout master`

- Merge pr branch to master
 	- `git merge pr-303`

- Ensure log shows that local master and pr branch are top, only repos are back one commit
 	- `git log`

- Push changes to origin master branch
 	- `git push origin master`
