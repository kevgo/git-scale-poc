# POC for scaling Git

This tutorial explores advanced Git techniques with a large mono-repo.
[Partial clones](https://docs.gitlab.com/ee/topics/git/partial_clone.html)
download only the content of files content that is actually used.
[Sparse checkouts](https://git-scm.com/docs/git-sparse-checkout) clone only
particular folders. For example, a developer on the UWT steelthread project
would clone only these folders:

- the folder containing the UWT code
- the folder containing shared libraries and dependencies
- the folder containing DevOps tools needed

They would not clone the other folders in the mono-repo, which contain for
example legacy apps, other steelthread code, etc.

We are going to use the
[source code of GitLab](https://gitlab.com/gitlab-org/gitlab) for this: a large
mono-repo with 2.8 GB of files, 177k commits, and 4,600 branches!

### Requirements

Before we get started, please make sure your machine has these requirements:

- a modern Git implementation: `git --version` should return `2.26` or better
- Java version 8 or better
- [Google Bazel](https://docs.bazel.build/versions/3.2.0/install.html)

### Initial clone

Let's get the initial clone! We are only downloading metadata and the content of
the files in the root directory:

```
git clone --filter=blob:none --sparse git@gitlab.com:gitlab-com/www-gitlab-com.git
```

Since we download only a shell of the actual repository, no more than 300MB
instead of 2.8 GB, this took only about a minute.

Let's explore the repo:

```
cd gitlab
```

This folder contains only the files in the home directory. Let's initialize the
sparse checkout:

```
git sparse-checkout init --cone
```

Let's say we want to work on the documentation for the Telemetry feature. Let's
add that folder:

```
git sparse-checkout add doc/development/telemetry
```

This took just a few seconds. Now we get all the files in the folder
`doc/development/telemetry` and all its parent folders. You can open
`doc/development/telemetry/index.md` and make some changes. Save them and run
`git status`. You see 1 changed file. The operation is instantaneous. Let's get
updates:

```
git pull
```

This takes just a few seconds.

### Building with Bazel

- Java library
- Java micro-service
- TypeScript library
- Angular app
- Python app

### future work

- setting up multiple
  [promisor remotes](https://git-scm.com/docs/partial-clone#_using_many_promisor_remotes)
