---
layout: post
date: 2022-05-17
categories: tips, git, gpg
title: Import and trust a public key
---

To import a public key from another user so you can verify signed git commits.

1. Download the public key
2. Import key into gpg `gpg --import {keyfile}`
3. List key to get id `gpg --list-keys`
4. Edit key `gpg --edit {keyId}`
5. Trust the key `trust`
6. Set the level of trust to 5 `5`
7. Clost gpg `quit`

You can now verify the git commits.

`git verify-commit {comit}`

`git log --show-signature`
