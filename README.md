example-go-app
==============

This is a follow-along example of my preferred way to maintain
dependencies for a Go (golang) project using git subtree.

Introduction
============

There are many ways to build and maintain a Go project that requires
external libraries.

The simplest way (which is already built into the Go toolchain itself)
is to just run `go get` on all your dependancies, and hope that the
library authors don't ever introduce breaking changes into their code. 
For small and personal projects that is fine, but for people who are
writing code for a living, that isn't nearly good enough.

There are many tools that have sprouted up among the Go community for
managing library dependencies.  You can visit the [Go
Wiki](https://code.google.com/p/go-wiki/wiki/PackageManagementTools)
for some of the popular ones.

However, for my work, I don't wish to use any of those tools, and
furthermore I want to use a dependency management scheme that is not
Go-specific.

My DVCS of choice is [`git`](http://www.git-scm.org), and recent
versions of git now have the `[git
subtree](http://blogs.atlassian.com/2013/05/alternatives-to-git-submodule-git-subtree/)`
command which makes working with separate library projects smoother than
before (such as with using the `git submodule` command).

This project is intended to be an example of my preferred workflow
with Go and `git subtree` which makes things relatively easy for the
other developers on my team, and still allows us to contribute changes
back to the upstream authors.

Development Scenario
====================

We're going to be writing a small application for our company
'example.com', and we're going to need a couple different libraries for
this project.

In one case, with the [upsilon
project](https://github.com/jamesgraves/upsilon), we don't anticipate
needing to make any major changes to the library, we will definitely
want to stay up-to-date with improvements to it, and we will probably
just be contributing bugfixes back to the upstream author.

The other library is
[omicron](https://github.com/jamesgraves/omicron), and we've decided
to fork it, because we're planning on making incompatible changes to
it.

I've already forked these two projects on GitHub, and if you're
following along, you will want to do the same under your own account.
Of course you'll then need to adjust the repository paths that follow.


Initial Checkout
================


```
$ mkdir myproject
$ cd myproject
$ git clone git@github.com:jamesgraves/example-go-app.git
Cloning into 'example-go-app'...
[...]
Checking connectivity... done.
```

We're going to be using a per-project GOPATH.  This is because the
entire project is under version control, and we want to be sure we've
got the exact version of every file that goes into the build.

We really can't do things like set GOPATH to `${HOME}` as is often
suggested, because we want to make sure all the source code is in the
main project repository.


```
$ cd example-go-app/
$ export GOPATH=~/myproject/example-go-app 
```

Let's switch to a new branch so that we won't mess up the master
branch used for this example.


```
$ git checkout working
Branch working set up to track remote branch working from origin.
Switched to a new branch 'working'
$ ls src/example.com/myapp/
example_main.go
```

In this case, I had already created a `working` branch, so you'd
normally need to run checkout with the `-b` option to create the
branch.

Let's edit the initial main program, and run it.

```
$ vi src/example.com/myapp/example_main.go
$ cat src/example.com/myapp/example_main.go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("This is main.")
}
$ go run src/example.com/myapp/example_main.go
This is main.
```

This is all standard git stuff so far.

Import Libraries into Our Project
=================================

Now it is time to add the remote for the library we're going to fork
and maintain ourselves.

```
$ git remote add -f omicron-vendor git@github.com:jamesgraves/omicron.git
Updating omicron-vendor
warning: no common commits
[...]
From github.com:jamesgraves/omicron
 * [new branch]      master     -> omicron-vendor/master
 * [new branch]      working    -> omicron-vendor/working
```

Don't worry about the 'no common commits' that is to be expected.

As before, I had already created 'working' branches, so you may want
to do that first.  Otherwise, you can push / pull from master in all
the following examples.

We can now add the library as a subtree in our existing project.

Since we're going to fork this library, we're going to put it in our
project directory under our own path:

```
$ mkdir -p src/github.com/jamesgraves/
```

And now bring in the actual source.  It isn't necessary to squash the
commits, but it may make it easier to read the history.

```
$ git subtree add --prefix src/github.com/jamesgraves/omicron omicron-vendor master --squash
git fetch omicron-vendor master
From github.com:jamesgraves/omicron
 * branch            master     -> FETCH_HEAD
Added dir 'src/github.com/jamesgraves/omicron'
$ find src -type f -print
src/example.com/myapp/example_main.go
src/github.com/jamesgraves/omicron/LICENSE
src/github.com/jamesgraves/omicron/.gitignore
src/github.com/jamesgraves/omicron/README.md
src/github.com/jamesgraves/omicron/lib.go
```

You may note that there is no .git directory in there, everything is
stored in the main .git directory.

The omicron library is pretty basic:

```
$ cat src/github.com/jamesgraves/omicron/lib.go
package omicron

import ( "fmt" )

func Foo() {
	fmt.Println("This is Foo() from package omicron")
}
```

For the upsilon library that we're only going to make bugfixes to,
we're not going to change the original directory heirarchy.  So we
aren't going to change location for where it lives in our project
directory.  It is going to end up in the same place as if we had just
run `go get github.com/upstream-author/upsilon` (if upstream-author
actually existed).

```
$ mkdir -p src/github.com/upstream-author
```

Add the remote of our fork.

```
$ git remote add -f upsilon-vendor git@github.com:jamesgraves/upsilon.git
Updating upsilon-vendor
warning: no common commits
[...]
From github.com:jamesgraves/upsilon
 * [new branch]      master     -> upsilon-vendor/master
```

And pull in its source too.

```
$ git subtree add --prefix src/github.com/upstream-author/upsilon upsilon-vendor master --squash
git fetch upsilon-vendor master
From github.com:jamesgraves/upsilon
 * branch            master     -> FETCH_HEAD
Added dir 'src/github.com/upstream-author/upsilon'
$ ls src/github.com/upstream-author/upsilon
lib.go	LICENSE  README.md
```

As with the omicron library, upsilon is also somewhat unambitious.

```
$ cat src/github.com/upstream-author/upsilon/lib.go
package upsilon

import ( "fmt" )

func Bar() {
	fmt.Println("This is Bar() from package upsilon")
}
```

Let's now edit our main program to use these new and fantastically
complicated libraries.

```
$ vi src/example.com/myapp/example_main.go
$ cat src/example.com/myapp/example_main.go
package main

import (
	"fmt"
	"github.com/jamesgraves/omicron"
	"github.com/upstream-author/upsilon"
)

func main() {
	fmt.Println("This is main.")
	omicron.Foo()
	upsilon.Bar()
}
$ go run src/example.com/myapp/example_main.go
This is main.
This is Foo() from package omicron
This is Bar() from package upsilon
```

Whew!  That was a lot more work than just running `go get`.

Edit and Push Changes to Libraries
==================================

Let's push these imported libraries into our central project
repository to the 'working' branch.

```
$ git status
# On branch working
# Your branch is ahead of 'origin/working' by 4 commits.
#   (use "git push" to publish your local commits)
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   src/example.com/myapp/example_main.go
#
no changes added to commit (use "git add" and/or "git commit -a")
$ git commit -m 'Now using omicron and upsilon libs' src/example.com/myapp/example_main.go 
[working 926bda2] Now using omicron and upsilon libs
 1 file changed, 4 insertions(+)
$ git push origin working
Counting objects: 25, done.
[...]
Total 25 (delta 4), reused 0 (delta 0)
To git@github.com:jamesgraves/example-go-app.git
   e1d5241..926bda2  working -> working
```

Time to make a small bug fix for upsilon, and push that back to our
own example project.

```
$ vi src/github.com/upstream-author/upsilon/lib.go 
$ cat src/github.com/upstream-author/upsilon/lib.go 
package upsilon

import ( "fmt" )

func Bar() {
	fmt.Println("This is Bar() from package upsilon, with a small bugfix.")
}
git commit -m 'Bugfix for print.' src/github.com/upstream-author/upsilon/lib.go
[working c4d49fb] Bugfix for print.
 1 file changed, 1 insertion(+), 1 deletion(-)

git push origin working
Counting objects: 7, done.
[...]
To git@github.com:jamesgraves/example-go-app.git
   926bda2..c4d49fb  working -> working
```
Note that we haven't pushed our changes to back to the library project
itself, just to our application project.

Since that won't really help the other users of the upsilon library
(because our main project may not be public), we need to push that bug
fix back to our fork on upsilon.  

```
$ git subtree push --prefix=src/github.com/upstream-author/upsilon upsilon-vendor working
git push using:  upsilon-vendor working
[...]
To git@github.com:jamesgraves/upsilon.git
   e8ffa07..f920c4f  f920c4f0d3e9df6b3fc52988a192fd39cfa72bb4 -> working
```

So this is going to push the f920c4f back to our fork at
github.com:jamesgraves/upsilon, and from there we can open a pull
request with upstream-author for this bug fix.

The situation will be similar for adding new functionality to our fork
of the omicron library.

```
$ vi src/github.com/jamesgraves/omicron/lib.go
$ cat src/github.com/jamesgraves/omicron/lib.go
package omicron

import ( "fmt" )

func Foo() {
	fmt.Println("This is Foo() from package omicron.")
	fmt.Println("Major new functionality for omicron.");
}
$ git commit -m 'New functionality.' src/github.com/jamesgraves/omicron/lib.go
[working 6f8ced7] New functionality.
 1 file changed, 2 insertions(+), 1 deletion(-)
$ git push origin working
Counting objects: 7, done.
[...]
To git@github.com:jamesgraves/example-go-app.git
   c4d49fb..6f8ced7  working -> working
```

And finally we can push our new functionality to our fork of omicron
so that other users of the omicron library can easily get it too.

```
$ git subtree push --prefix=src/github.com/jamesgraves/omicron omicron-vendor working
git push using:  omicron-vendor working
[...]
To git@github.com:jamesgraves/omicron.git
   5808bb0..ef181b9  ef181b96c7155e126310df11ac2b887d7747e9c0 -> working
```

Since it runs, ship it! (Or at least tag it for 'alpha'.)

```
$ git tag alpha_release
$ git push --tags origin
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:jamesgraves/example-go-app.git
 * [new tag]         alpha_release -> alpha_release
```

If we later see an update to upsilon, we can pull that to our GitHub
fork, and then pull it into our project.

```
$ git subtree pull --prefix src/github.com/upstream-author/upsilon
upsilon-vendor working --squash
```
You'll be offered to edit the merge commit message.

Note that for the `--prefix` above, we have to specify the directory
upsilon is in.  Before with the `subtree add`, the `--prefix` used
with that command specified the parent directory where upsilon should go.

Conclusion: That was waaaayyyy too much work!
=============================================

OK, so if you're the one who's worried about contributing changes back
to the upstream authors, there is a bit of initial effort to set up
the new remotes and the commands to push and pull.

And by now you're waiting for me to at least provide a fancy, smancy
script that will automate all of the above.  Hah!  In your dreams!

There is a payoff though, and a big one in my opinion.  You've just
saved a lot of work for the fellow developers on your team, and you've
made life easy for QA and release engineering too.  Here's what they
do to check out the alpha release:

```
$ cd ~/
$ mkdir new_developer_workspace
$ cd new_developer_workspace/
$ git clone git@github.com:jamesgraves/example-go-app.git
Cloning into 'example-go-app'...
remote: Counting objects: 50, done.
[...]
Checking connectivity... done.
$ cd example-go-app/
$ export GOPATH=~/new_developer_workspace/example-go-app
$ git checkout alpha_release
Note: checking out 'alpha_release'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 6f8ced7... New functionality.

```
That's all standard git stuff.

And now they can just run the app, continue developing, and
everything, all without having to worry about the other external
libraries.

```
$ git remote show origin
* remote origin
  Fetch URL: git@github.com:jamesgraves/example-go-app.git
  Push  URL: git@github.com:jamesgraves/example-go-app.git
  HEAD branch: master
  Remote branches:
    master  tracked
    working tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
$ go run src/example.com/myapp/example_main.go
This is main.
This is Foo() from package omicron.
Major new functionality for omicron.
This is Bar() from package upsilon, with a small bugfix.
```
Everyone else on the team is just going to be pushing / pulling from
origin, and that's that.

As long as no one makes a commit that crosses library boundaries,
you'll have an easy time picking out changes that can be pushed back
to upstream.  And you'll be able to pull in fixes for the libraries
you use.

QA and release engineering should be happy because your build is 100%
reproducible.  And they don't have to learn any new tools or
procedures.  Just check out a particular tag, build and Go!

Final Thoughts
==============

I wrote this mostly as a response to people that insist that Go must
have some means of doing dependency management.  I don't think Go
needs to implement something like that, and other tools are better
suited to the task.

As always, if you see any errors or problems with this, please feel
free to send me fixes.


If you've found
