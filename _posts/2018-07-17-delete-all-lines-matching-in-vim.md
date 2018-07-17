---
layout: post
date: 2018-07-17
categories: vim
title: How to delete all lines matching a pattern in VIM
---

Sometimes you need to delete a number of similar lines in a file. Instead of deleting each line individually, if you are using vim or vim like editor you can execute the following command.

```
:g/{pattern}/d
```

This will delete all lines matching the pattern you specify.