Managing multiple dependent repositories with Git
=================================================

This project:
* helps you manage multiple Git repositories with complex dependencies
* provides a new git command 'git dep'
* is a greatly improved version of 'git submodule'

One of Git best practices is to keep separate projects in separate repositories.
Unfortunately Git does not make it easy to keep track of multiple repositories
with complex dependencies between them.

An real-life example we'd like to give you is a set of 6 repositories:
* C(ommons), containing common components to multiple projects,
* L(ibraries), containing common external dependencies,
* D is a common external component,
* S depends on C, L and D,
* E depends on S, D, C and L.
* T depends on C and L.

It is important for each commit of E to save information about corresponding
commits in S, D, L and C. Otherwise the history will be unusable, because
compiling a historic version of E would be require long and error-prone 
digging in its dependencies to determine the versions to check out.

The same applied to other repositories with dependencies. Commits of S should
carry information about C, L and D versions. Commits of T should point to 
suitable commits in C and L.

Solving this problem is impossible with git-submodule, first because it
requires the dependent repository to be checked out physically inside 
the parent one (which is impossible because there are duplicate dependencies),
second because it's command-line interface is not friendly enough for active
development (e.g. it tends to leave the HEAD detached).


Using git-dep
-------------

Dependencies information is kept in .gitdeps file in the root of a repository.

Additionally, each repository is given a unique name (e.g. C), and is referred to
by other repositories using this name.

Running 'git dep init' is necessary in each repository before it is usable as
a dependency or a dependent. Init command assigns a name to the repository and
stores its name and path in ~/.gitconfig. (Specify the name on the command line,
default is the name of the directory.)


Start using git-dep:

~/c% git dep init
~/c% git add .gitdeps
~/c% git commit -m "Started using git-dep"

~/c% cd ../e
~/e% git dep init
~/e% git dep add ../c
~/e% git add .gitdeps
~/e% git commit -m "Started using git-dep"


Cloning:

~% git clone ssh://.../e.git
~% cd e
~/e% git dep init
~/e% git dep clone

All dependencies (s.git, d.git, c.git and l.git) are cloned automatically.

Note: origin URL of dependencies is recorded when they are added and when their commit id
is updated. Make sure you have remote 'origin' in all your repositories for 'git dep clone'
to work.


Checking out another branch:

~/e% git checkout another-branch
~/e% git dep checkout

A correct branch is automatically checked out in all repositories we depend on.

Default behaviour of 'git dep checkout' is to checkout the correct branch if it
descends from the commit recorded in the current repository. This is a 'development'
least-surprise mode.

Note that 'git dep checkout' never changes the branch. If the tip of the branch
is not suitable, a new branch will be created with the correct commit.


Checking out a historical version:

~/e% git checkout 121OLDVERSION78374BBAA
~/e% git dep checkout -o

A correct commit is automatically checked out in all repositories we depend on.
A new branch is created in each repository to prepare for some maintenance work.
If the commit was originally on branch master, the created branch is named master-oldver.

Option -o (--old) changed the behaviour of 'git dep checkout' so that it stopped considering
descendant commits to be suitable, and thus it checked out the recorded commit exactly.

Again, 'git dep checkout' will never trash any work done on any branch, including the
newly created '-oldver' one. It may however fail to checkout the necessary revision
if there are new commits on the branch (and thus it cannot be reset).

All these cases are diagnosed with carefully worded messages, so no misunderstanding
should occur.


Commiting a change:

~/c% git commit ...
~/c% git dep record

Any repositories that depend on C are automatically updated with the new commit id and the current
branch name.


Alternative:

~/c% git commit ...
...
~/e% git dep record

This updates E's dependency information with the current commit id and branch name of S, D, C and L.

('git dep record' both updates repositories that depend on the current one, and updates the current
repository with actual information about its dependencies.)


Obtaining a list of dependencies:

~/e% git dep    (or: git dep status)


Obtaining a list of all repositories of the current user:

~/e% git dep all


Hooks
-----

The future plans include making 'git dep init' set up local hooks, so that:

* 'git dep checkout' is automatically executed on 'git checkout',
* 'git dep record' is automatically executed on 'git commit',
* dependent repositories are checked for uncommitted changes before 'git commit'.

Right now, however, you have to run these commands manually, and make sure to commit changes in dependee
repositories before commiting the dependent ones.


Maintainer and license
----------------------

This script is licensed under GNU General Public License v2 (or any newer version at your preference).

It is maintained by Andrey Tarantsov. Report bugs to andreyvit@gmail.com.
