# Scalable Git for developers

This experiment examines modern Git features for large code bases using the
[GitLab codebase](https://gitlab.com/gitlab-org/gitlab) as an example. Over 4000
developers contribute to this public monorepo, creating up to a few hundred
commits per day. It contains 1.2 GB of content, 33,000 files, 177,000 commits
from 9 years, and in total takes around 4 minutes to clone. Utilizing
[Sparse checkouts](https://git-scm.com/docs/git-sparse-checkout) and
[partial clones](https://docs.gitlab.com/ee/topics/git/partial_clone.html), the
initial clone of the repo can be reduced to 58 files and 300 MB of payload,
which downloads in about a minute. The user can add files and directories needed
for development one by one, their content will be lazy-loaded from the
repository. In the future, there will be support to stream content from multiple
promisor remotes.

### Requirements

- Git version 2.26 or later
- a computer with at least 8, ideally 16 GB RAM

### Initial clone

Clone only the metadata, branches, and the files in the root directory:

```
git clone --filter=blob:none --sparse git@gitlab.com:gitlab-org/gitlab.git
```

Explore the repo:

```
cd gitlab
```

This folder contains only the files in the home directory. Let's initialize the
cone pattern set for reasonable behavior and better performance:

```
git sparse-checkout init --cone
```

### Adding a folder

Add a particular subfolder we want to work on, in this case the documentation
folder for the Telemetry feature:

```
git sparse-checkout add doc/development/telemetry
```

This took just a few seconds. Now we get all the files in the folder
`doc/development/telemetry` and all its parent folders.

You can open `doc/development/telemetry/index.md` and make some changes. Save
them and run `git status`. You see 1 changed file. The operation is
instantaneous. Let's get updates:

```
git pull
```

This takes just a few seconds.

More information: https://docs.gitlab.com/ee/topics/git/partial_clone.html

### Code search

A full-text search for `potato` using [RipGrep](https://github.com/BurntSushi/ripgrep) on Linux takes 700 milliseconds:

```
time rg potato

(7 results)

Executed in 785 millis
```

A full-text search for `banana` using `git grep` on Linux takes only 400 milliseconds:

```
time git grep banana

(96 results)

Executed in 464 millis
```

### Gathering statistics

This experiment used the following methods to gather statistics.

- count total commits: `git log --oneline | wc -l`
- count commits per day: `git log --pretty=format:"%cs" | sort | uniq -c | sort`
- count storage used: `du -hd1`

### What this mean for Fannie Mae

Modern Git can handle even sizeable code bases with large amounts of history and
ongoing development traffic comfortably without any tricks. For repositories
that contain even larger amounts of data, there are capable techniques that
allow to download only subsets of the repository and its history onto the
developer machine. For example, a developer on the UWT steelthread project would
clone only these folders:

- the folder containing the UWT code
- the folder containing shared libraries and dependencies
- the folder containing DevOps tools

They would not clone the other folders in the mono-repo, which contain for
example legacy apps, other steelthread code, etc.
