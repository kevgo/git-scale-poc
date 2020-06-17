# Git Scalability Experiments

This experiment examines modern Git features for large code bases. 
As a reference point we use the
[GitLab codebase](https://gitlab.com/gitlab-org/gitlab). Over 4000
developers contribute to this public monorepo, creating up to several hundred
commits per day. It contains 1.2 GB of content, 33,000 files, 177,000 commits
from 9 years.

To run these experiments, you need Git version 2.26 or later.

### Sparse checkouts

A complete clone of all 1.2 GB and 33,000 files in the GitLab codebase takes around **4 minutes**. 

Utilizing
[Sparse checkouts](https://git-scm.com/docs/git-sparse-checkout) and
[partial clones](https://docs.gitlab.com/ee/topics/git/partial_clone.html), the
initial clone of the repo can be reduced to a payload of **58 files** and **300 MB**,
which downloads in about **one minute**. 

```
git clone --filter=blob:none --sparse git@gitlab.com:gitlab-org/gitlab.git
```

The user can then add files and directories needed for development one by one, 
their content will be lazy-loaded from the repository. 
In the future, there will be support to stream content from multiple promisor remotes.

Initialize the "cone" strategy for partial clones:

```
git sparse-checkout init --cone
```

Adding a folder and downloading its content takes only a few seconds:

```
git sparse-checkout add doc/development/telemetry
```

### Making changes 

Running `git status` after making changes to a few files is instantaneous.

### Pulling updates

After a day or two, there have been a lot of updates to the `master` branch.
Pull down updates takes no more than **20 seconds**:

```
git pull
```

More information at https://docs.gitlab.com/ee/topics/git/partial_clone.html

### Code search

Git comes with capable built-in full-text search capabilities. 
It finds all occurrences of the word `banana` in the entire 1.2 GB and 33,000 files in only **400 milliseconds**:

```
time git grep banana

(96 results)

Executed in 464 millis
```

A more capable full-text search tool is [RipGrep](https://github.com/BurntSushi/ripgrep). 
It finds all occurrences of the word `potato` in **700 milliseconds**:

```
time rg potato

(7 results)

Executed in 785 millis
```

[VSCode](https://code.visualstudio.com) is an IDE written in JavaScript. 
It finds all 103 occurrences of the word `banana` in 1.5 seconds.

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
