---
layout: post
title: "SSH agent forwarding and Github"
date: 2012-04-20 08:02
comments: true
categories:
---

[Github's help](http://help.github.com/deploy-keys/) says,

> "If your deploy process involves sshing into the server you are deploying to, you probably do not need to use deploy keys. Instead, you should use ssh-agent forwarding to temporarily allow the server to use your local ssh keys. Not only is this method easier to maintain, since you don’t have any extra keys, but it’s also more secure as the server never has keys saved to disk in case of a compromise."

But how to use ssh-agent fowarding?

It's pretty simple actually, but I made it work after some research.

First, ssh-agent forwarding can be enabled by,

* modifying the config file

    Host *
      ForwardAgent yes

* using -A option in the command line.

After enabling ssh-agent forwarding, I can connect another server from a connected server without sending a password. The private key on my MacBook is used for authentication.

But when I tried to run "sudo git pull", the command failed. In order to use ssh-agent forwarding after running sudo, I need to keep the environment variable `SSH_AUTH_SOCK`. This could be done by editing /etc/sudoers file,

    Defaults    env_reset
    Defaults    env_keep = "SSH_AUTH_SOCK"

After that, I can use git commands smoothly :)
