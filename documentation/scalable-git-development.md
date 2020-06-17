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
Initialize the "cone" strategy for partial clones:

```
git sparse-checkout init --cone
```

Add a folder and downloading its content takes only **a few seconds**:

```
git sparse-checkout add doc/development/telemetry
```

### Making changes 

Running `git status` after making changes to a few files is instantaneous.

### Pulling updates

- pulling down thousands of updates after a week of Gitlab development: **20 seconds**
- pulling down several weeks of updates (50 MB in total) for the Linux kernel: **50 seconds**


More information at https://docs.gitlab.com/ee/topics/git/partial_clone.html

### Code search

Git comes with built-in full-text search capabilities. 
It finds 96 occurrences of the word `banana` in the entire 1.2 GB and 33,000 files of the GitLab codebase in only **400 milliseconds**:

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
