# Git tips and tricks

  Git is the most popular version control system now and for the foreseeable future. At the same time, most often developers use only the possible minimum of possibilities, since the Git commands are quite complex, and the documentation is verbose and covers all the features and does not suggest an optimal way to solve typical problems, nor does it demonstrate the unobvious useful features of this system. Here I want to share a very small set of commands, which is enough for me to completely control how my Git repositories look, and also solve 99% of the problems that arise.

* [Commit](#commit)
  * [What is commit?](#what-is-commit)
  * [How ideal commit looks like?](#how-ideal-commit-looks-like)
  * [Why simplest approach is not optimal?](#why-simplest-approach-is-not-optimal)
  * [How to make a good commit?](#how-to-make-a-good-commit)
* [Rewriting history](#rewriting-history)
  * [Change last commit](#change-last-commit)
  * [Change some commits on current branch](#change-some-commits-on-current-branch)
* [Flow](#flow)
  * [Master only](#master-only)
  * [Multibranch, merge only](#multibranch-merge-only)
  * [Multibranch, rebase and merge](#multibranch-rebase-and-merge)
  * [Git flow](#git-flow)
* [Issues](#issues)
  * [Reflog](#reflog)
  * [Cherry-pick](#cherry-pick)
* [Tips and tricks](#tips-and-tricks)
  * [Worktrees](#worktrees)
  * [Blame](#blame)
* [Other features](#other-features)
  * [Submodules and subtrees](#submodules-and-subtrees)
  * [Hooks](#hooks)
  * [Smart commits](#smart-commits)
* [List of useful commands](#list-of-useful-commands)
  
## Commit
 Commits are a core Git feature that cannot be avoided. There are many ways to commit code - via the command line or various GUI tools.
Here I would like to first discuss good practices and then show you how to practice them.

### What is commit?
 Commits are like milestones, they store a certain state of files. This has many useful uses.
* A commit is a backup. You can go back to it if you completely messed things up, or you can just see what exactly you changed before everything broke.
* A commit is a means of communication. The commit contains the changes, and other programmers can apply those changes on their own copy of the code.
* Commits are a history of changes.
* A commit can serve as a description of the code and its changes, that is, documentation.

In most cases, commits only serve as a means of automatic code exchange.

### How ideal commit looks like?
Imagine if each commit had only one change, such as a new feature, code formatting, bug fix, unused code removal, or a little local refactoring. Also, each commit would have a message that clearly indicated the change in the title and other useful information regarding the change in the description.
In this case, commits serve as a history of changes and documentation, and those who read your code, for example, new team members, can see when, how and why you made changes. Of course, you can just comment the code, but comments in the code get outdated, and the messages in commits do not.
In addition, if each of your commit has only one purpose, you are much less likely to introduce accidental errors, because all changes are extremely visual. It also helps your reviewers, as they can see changes separately and see if they make sense in a given context.

### Why simplest approach is not optimal?
Many Git users make commits like this:
```
git add.
git commit -m "feature 1 and began to work on feature 2. Also fixed some bugs"
```
Or they do it using GUI tools. Sometimes they select individual files, but without checking them entirely.

The problem here is that there is a high probability of committing something that is not related to any topic from the commit message, or even committing code that was added only temporarily (for test / debug)
Another problem is that such a change cannot be verified, it covers several different topics at the same time, it is not clear what change in the code is for what.

### How to make a good commit?
Whenever you feel like committing something, I recommend using Atlassian SourceTree (or a similar GUI tool) to select the changes you want to make.
You can select whole files, or parts, or lines individually. This ensures that you can only choose what is relevant. You can do this from the command line, but I wouldn't call the process visual or easy.
Then you can commit using the same GUI, or on the command line:
```
git commit
```
 A text editor will open in which you can type the commit message. In it you should enter a title - the name of your change, then a more detailed description. For trivial commits, no description is needed.
```
git commit -m "<commit message>"
```
If you are a one-line commit message and does not need a detailed description.

Perhaps after that there will be some other changes that need to be committed. To do this, you need to create a separate commit by repeating the process.

It's perfectly okay if instead of just one commit, you end up with 3, 5 or more. You will spend a little more time doing this, but you will be absolutely sure what exactly you changed.

## Rewriting history
If you've never tried to rewrite the history of commits, or think it's too much trouble, this part for you. You need to know no more than two commands to be able to change, remove, rename, merge, or add commits on the current branch.

### Change last commit
Changing the last commit is a little easier than changing any other. There is a special command:
```
git commit --amend
```
This command adds staged changes to the last commit.

If you do not want to change the commit message, but only add changes, then you can improve the command:
```
git commit --amend --no-edit
```

### Change some commits on current branch
The most important command in this process is
```
git rebase -i HEAD~<number of commits you are going to change>
```
eg
```
git rebase -i HEAD~10
```

#### Prerequisits
Ensure that you don't have uncommited changes. You can commit it or [stash](#stash)

#### Editing history 
When you run this command, you will be shown text similar to the following in a text editor:
// TODO picture

```
p, pick = use commit
r, reword = use commit, but edit the commit message
e, edit = use commit, but stop for amending
s, squash = use commit, but meld into previous commit
f, fixup = like "squash", but discard this commit's log message
x, exec = run command (the rest of the line) using shell
d, drop = remove commit
```
These are the options available, for each commit individually

Most often you need `pick`, `edit` and `fixiup`.

If you need to delete a commit, you can simply delete it, or use `drop`.

It is possible to reorder commits, but be careful, it can lead to conflicts if one change overlaps with another.

`pick` means just to take a specific commit, no change

`edit` explains itself.
`fixup` - adds the contents of a commit to the previous commit, and throws out the commit message.

After that just save changes and exit the editor

#### Command execution
After you save the changes and exit the text editor, the command will start re-adding those commits, in the order you specified. 
The command will interrupt for `edit` - if you chose this option for a commit, the text editor will open again, where you can edit the commit message.

Also, there may be a conflict of changes. In this case, you can try to resolve this conflict and then execute the command
```
git rebase --continue
```
to continue
or, if it doesn't make sense anymore, you can abort the whole process and return to the original state with the command
```
git rebase --abort
```

#### Fixups
Sometimes it happens that you are working on something, periodically adding commits. And then you notice that you need to add a certain change several commits back
You can commit this change with a message like "A fixup to commit <commitname>" and then manually move that commit to the correct location with git rebase -i`. 
  
But there is a slightly more convenient way
```
git commit --fixup=<commit hash>
```
When you run `git rebase -i`, the commit will be added to the correct location automatically, with `fixup` key.

#### Is it too difficult?
This command may look too difficult at first, but after a little practice, you may find that power of this command is worth troubles of learning it. It gives you complete power over the history of changes you make, and you can assemble commits to clearly represent your intentions, instead of chronological changes.

## Flow
There are several approaches to branching in Git. Some are chosen for workflows, while others come from ignorance of better options. Here I will try to consider the most popular options, their scope, advantages and disadvantages.

### Master only
This option is perhaps the most popular for repos with very small teams. 

The obvious advantage of this process is its simplicity. Everything can be done with the most primitive tools, right from the IDE, without even knowing a single Git command. And if only one person needs to work with the repository, then the history of changes does not branch at all.

The disadvantage of this approach is poor scalability and the fact that it motivates to use the worst practices.

Poor scalability manifests itself in an increasing number of parallel branches and merge commits. This makes it very difficult to retrospectively track changes.

And the worst practices are, for example, the inability to effectively rewrite the history of local changes due to the constant inclusion of merge commits in the history. It is also impossible to determine which topic the commits are related to. This creates a chaotic Git culture.

### Multibranch, merge only
This option means that at the beginning of work on each feature, a new branch is created from the latest version of master. At the end of the work, the branch is merged into master.

This option is often used when a team using the previous option is implementing a code review. This leads to the need to use pull requests, and as a result, named branches.

The advantage is obviously that it allows pull requests. It also allows you to understand what topic each commit belongs to.

The disadvantages are poor scaling too. Each developer can have multiple branches, and if there are many developers, this leads to an unreadable history of changes in Git.

### Multibranch, rebase and merge
This option means that at the beginning of work on each feature, a new branch is created from the latest version of master. At the end of the work, the branch is rebased to the latest version of master (the latest at this point) and then the branch is merged into master.
That is, it looks like that the branch was created, and was immediately merged into master. At the same time, the history of changes looks almost linear, with the exception for merge commits, which show the checkpoints at which the new feature entered master.

In fact, this option is not very popular, but rather reflects my positive experience.

The advantage of this approach is that you have a very clear history of changes in Git and can be confident that your changes will merge easily.

The downside is that if you have merge conflicts, it might be easier to do without it and just merge.

### Git flow
This is a fairly complex system, although very effective in many cases. It is better to read about it [here](https://nvie.com/posts/a-successful-git-branching-model/). The main idea, in comparison with the previous two options, is that instead of master, the develop branch is used. And only tested commits from the develop branch are merged into the master branch. So the process is two-stage. While this process may not be suitable for everyone, many of the practices of this process, such as naming branches, are still worth learning.

## Issues

### Reflog
This command allows you to fix bugs such as accidentally deleting a branch, commit, stash, or just about any other change.

It's pretty simple to use.
git reflog
The command will list the recent commands and the corresponding commit checksums. After that, you can recreate the branch.
```
git branch <branch_name> <commit_checksum>
```

### Cherry-pick
This simple command is to take any commit from anywhere to current branch.
```
git cherry-pick <commit_checksum>
```
Usually it is useful for picking some changes from failed branches.

## Tips and tricks

### Worktrees
Often it is necessary to work simultaneously with several branches of the same project. A possible solution is to just copy the folder. However, Git provides a better solution - the git worktree commands. These commands are used to manage different branches of the same repository in different folders, while remaining part of the same repository, with a single copy of the .git directory.

Example:
```
git worktree add <path> [<commit-ish>]
// Example
git worktree add ../Example_branch1 branch1
```
Show list of worktrees:
```
git worktree list
```
Remove a worktree:
```
git worktree remove ../Example_branch1

// Sometimes Git will not obey, because of some uncommited changes. If you don't need these changes anymore, add option -f
git worktree remove -f ../Example_branch1
```

### Blame
If you need to know who changed certain lines of a file and when, you can use GUI tools such as Git -> File history in Visual Studio or the Blame button on GitHub. But these tools only work in simple cases. If the file has been renamed, moved, copied, split or merged, these tools will only show the author and the time the file was created. Git has several commands that will untangle history in almost any case, regardless of moving code through files:
```
git blame <file> // This command will show you same result as GUI tools
git blame -C -M -w <file> // This command will ignore changes in whitespaces, and trace all code movements in different files
```
This command has many other options, for example, you can set a specific commit as a starting point.
In addition to this, you can use `git log -p <file>` with different options to see the history of changes.

## Other features

### Submodules and subtrees
If you need to include one repository in another, for example, include a third-party open-source library in your project, Git provides several ways:

More complex but clean - submodules. You can read more about this [here](https://www.atlassian.com/git/tutorials/git-submodule)

Simpler - subtrees. You can read more about this [here](https://www.atlassian.com/git/tutorials/git-subtree)

### Hooks
Hooks let you run scripts on specific events in Git. For example, you can run a linter before committing. You can read more about this [here](https://www.atlassian.com/git/tutorials/git-hooks)

### Smart commits
Smart commits allow you to manage issues in the issue tracker using a commit message. [An example](https://support.atlassian.com/bitbucket-cloud/docs/use-smart-commits/) how it may work.

## List of useful commands
```
git commit // Commit staged changes, input commit message interactively
git commit -m "<commit message>" // Commit staged changes, message is an argument
git commit --amend // Add staged changes to the last commit and edit its message
git rebase -i HEAD~<number of commits you are going to change> // Rewrite last n commits - edit messages, fixup, squash, reorder, delete
```
