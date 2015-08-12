---
layout: post
title: Contribute
category: dev
---
{% include JB/setup %}

Like other Apache projects, SINGA uses JIRA to track bugs, improvements and
other high-level discussions (adding new features).  Github pull requests are
used for implementation discussions, e.g., code review and code merge.

### JIRA format

* Provide a descriptive Title.
* Write a detailed Description. For bug reports, this should ideally include a
  short reproduction of the problem. For new features, it may include a design
  document.
* Set [required fields](https://cwiki.apache.org/confluence/display/SPARK/Contributing+to+Spark#ContributingtoSpark-JIRA)

### Pull Request

The work flow is

* Fork the [SINGA Github repository](https://github.com/apache/incubator-singa) to
your own Github account.

* Clone your fork, create a new branch (e.g., feature-foo or fixbug-foo),
 work on it. After finishing your job,
 [rebase](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) it to the
 current latest master and push commits to your own Github account (the new
 branch).

* Open a pull request against the master branch of apache/incubator-singa.
The PR title should be of the form SINGA-xxxx Title, where
SINGA-xxxx is the relevant JIRA number, and Title may be the JIRA's title or a
more specific title describing the PR itself, for example, "SINGA-6 Implement
thread-safe singleton". Detailed description can be copied from the JIRA.
Consider identifying committers or other contributors who have worked on the
code being changed. Find the file(s) in Github and click "Blame" to see a
line-by-line annotation of who changed the code last.  You can add @username in
the PR description to ping them immediately.
Please state that the contribution is your original work and that you license
the work to the project under the project's open source license. Further commits
(e.g., bug fix) to your new branch will be added to this pull request
automatically by Github.

* Wait for one committer to review the patch. If no conflicts, the committers
 will merge it with the master branch. The merge should a) not use rebase b)
 disable fast forward merge c) put the SINGA-xxx title in the first line of
 the merge commit message.
