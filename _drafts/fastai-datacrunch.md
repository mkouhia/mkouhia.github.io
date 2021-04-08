---
title: fast.ai model development on DataCrunch.io
tags: AI, fast.ai, datacrunch.io, linux, server 
---

# fast.ai model development on DataCrunch.io

For developing and training deep learning models, a GPU-enabled computing resource is a must.
I did not even bother trying on my laptop, but went straight ahead to using cloud resources as recommended by
[fast.ai tutorial](https://course.fast.ai/). I started first using [Paperspace Gradient](https://gradient.paperspace.com/)
as per [fast.ai instructions](https://course.fast.ai/start_gradient),
but it soon occurred to me that the free notebook servers were most of the time loaded full.
Paying a bit extra would not have been a problem, but in order to access the paid GPU servers, a monthly subscription would
have been a must. I could not justify that cost yet, so I arrived at another suggested solution, [DataCrunch.io](https://datacrunch.io/)

Now, the good side with Paperspace Gradient was that you could start right where you left the last time.
DataCruch servers are paid by hourly basis, and at the time of writing persistence of sessions was not working.
Thus, using DataCrunch required starting each session from scratch every time.

I'm going to outline here some basics that enable streamlined development for me, using `ssh`, ssh key authentication and `git`.

## Setting up services, repositories and keys (one-time only)

### GitHub repository

https://github.com/settings/keys

### DataCrunch.io

https://course.fast.ai/start_datacrunch

### SSH keys

- Generating
- Adding to GitHub https://github.com/settings/keys

### Windows subsystem for Linux (WSL) ssh-agent

- https://dev.to/levivm/how-to-use-ssh-and-ssh-agent-forwarding-more-secure-ssh-2c32
- https://docs.github.com/en/developers/overview/using-ssh-agent-forwarding
- https://www.scivision.dev/ssh-agent-windows-linux/
- https://unix.stackexchange.com/a/388712

## Steps for each session

1. Login to DataCrunch via website, https://cloud.datacrunch.io/signin
2. Deploy an new server, https://cloud.datacrunch.io/dashboard/deploy-server
    - Get IP
4. Connect from WSL shell https://datacrunch.io/docs/connecting-via-ssh/

    ```sh
    $ ssh-add -L
    $ ssh -A user@<ip>
    ```
5. Copy resources to DataCrunch server instance
    ```sh
    $ git clone
    ```
5. Switch to web UI, start Jupyter notebooks, start developing

## At the end of session

1. Get exported files
    - Download to local computer with scp
    - Upload to cloud, example: Dropbox https://github.com/dropbox/dbxcli
2. *Delete* DataCrunch server

