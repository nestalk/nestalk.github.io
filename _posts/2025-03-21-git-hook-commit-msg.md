---
layout: post
date: 2025-03-21
categories: tips, git, tickets
title: Automatically adding ticket number to git commit message
---

It is very useful to tag commit messages with the ticket number it relates to, future you will thank you so much when he is trying to figure out what you were doing. A good way to automatically add the ticket number is by using a git `commit-msg` hook to pull the ticket number from elsewhere and add it to the commit message. I have standardised my branch names to include the ticket number, e.g. for ticket ABC-123 I will create a branch called `feature/ABC-123/descriptiveName`, so all that is required is to pull the ticket number from the branch.

To do this I have created a git hook to do that:

```BASH
#!/bin/bash

BRANCH="$(git rev-parse --abbrev-ref HEAD)" # get current git branch
[[ "$BRANCH" =~ [A-Z]+-[0-9]+ ]]            # regex to get the jira issue
ISSUE=$BASH_REMATCH                         # rename result of regex
if [ -n "$ISSUE" ]; then                    # only update if there is a issue
	sed -i "1s/^/[$ISSUE] /" $1             # add issue to commit message
fi
```

Once is save this script to a file called `commit-msg` in the `.git/hooks` folder of my local repository, every time I commit the message will be updated with the ticket number from the current branch.

Only issue is that it is only installed in the local git repository, you can't install it globally for your whole team. In order to share it with the rest of the team you need to add it elsewhere in the repo and ask your teammates to add the hook to their local hooks folder.  

This will work in all git clients and in Windows and Linux. It doesn't work on OSX for some reason, I don't use a mac so I have not looked into why.