---
title: fast.ai model development on DataCrunch.io
tags: AI, fast.ai, datacrunch.io, linux, server 
layout: post
---

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

Every command outlined is performede either in Linux environment (cloud servers) or Windows Subsystem for Linux (WSL).

### SSH keys

For remote access into the servers and repositories, set up SSH keys. [Here](https://www.ssh.com/academy/ssh/keygen) is an exhaustive
guide, for me it boils down to
```sh
$ ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```
as suggested over [here](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54).

### GitHub repository

I will be storing the notebooks and model in a GitHub repository. Each session will end up with results written in a notebook,
or some work that is done in a feature branch. 

For git-based workflows in data science, see e.g. [this essay](https://ericmjl.github.io/essays-on-data-science/workflow/gitflow/).

In GitHub, you can associate SSH keys with the user account at https://github.com/settings/keys. Add the key over there to enable
git access over ssh.

### DataCrunch.io

[DataCrunch.io](https://datacrunch.io/) provides cheap on-demand computing servers with GPU resources.
The fast.ai course provides a good summary of how to set up an account, see [here](https://course.fast.ai/start_datacrunch).
The step 3 in the guide considers creating an SSH key, we already did that. For step 4, read key from `~/.ssh/id_ed25519.pub`.


### Windows subsystem for Linux (WSL) ssh-agent forwarding

To allow SSH key forwarding, that is accessing GitHub on the remote server, while the private SSH key only resides on local computer,
configure `ssh-agent`.

For WSL, add following to `~/.bashrc` (reference: [SciVision](https://www.scivision.dev/ssh-agent-windows-linux/)) 
```sh
if [ -z "$(pgrep ssh-agent)" ]; then
   rm -rf /tmp/ssh-*
   eval $(ssh-agent -s) > /dev/null
else
   export SSH_AGENT_PID=$(pgrep ssh-agent)
   export SSH_AUTH_SOCK=$(find /tmp/ssh-* -name agent.*)
fi
```

Then either start a new WSL window or run `. ~/.bashrc`.

One can add SSH keys to `ssh-agent` with
```sh
$ ssh-add -t 3600 ~/.ssh/id_ed25519
Enter passphrase for /home/mkouhia/.ssh/id_ed25519:
Identity added: /home/mkouhia/.ssh/id_ed25519 (mkouhia@hostname)
Lifetime set to 3600 seconds
```
Here, parameter `-t life` sets how many seconds the identity is stored with the agent.

Now, you can verify whether you can access github with the stored key:
```sh
$ ssh -T git@github.com
Hi mkouhia! You've successfully authenticated, but GitHub does not provide shell access.
```

More reading:
- https://dev.to/levivm/how-to-use-ssh-and-ssh-agent-forwarding-more-secure-ssh-2c32
- https://docs.github.com/en/developers/overview/using-ssh-agent-forwarding
- https://www.scivision.dev/ssh-agent-windows-linux/


## Steps at the start of each session

1. Login to DataCrunch via website, https://cloud.datacrunch.io/signin
2. Deploy an new server, (see [guide](https://cloud.datacrunch.io/dashboard/deploy-server)) and take note of its **IP address**.
   Remember to check the checkbox next to the SSH public key you have added to DataCrunch. This will allow SSH access to the server
   with the key you have on the local computer.
3. Connect from local Windows WSL shell

    On local computer, add SSH key to agent and connect to server with SSH agent forwarding (the flag `-A`), with the actual username _user_
    ```sh
    $ ssh-add -t 3600 ~/.ssh/id_ed25519
    $ ssh -A user@<IP address>
    ```
    Now you can check that the SSH agent is forwarding the identity to GitHub
    ```sh
    $ ssh -T git@github.com
    Hi mkouhia! You've successfully authenticated, but GitHub does not provide shell access.
    ```
4. Clone working repository from GitHub to DataCrunch server instance, using the authentication key provided by SSH agent
    ```sh
    $ git clone <repository address>
    ```
5. _(optional)_ If you uploaded data to Dropbox or someplace else at the end of previous session, download and extract them in the repository folder

    ```sh
    $ cd <repository folder>
    $ curl -L https://github.com/dropbox/dbxcli/releases/download/v3.0.0/dbxcli-linux-amd64 -o dbxcli
    $ ./dbxcli account
    1. Go to https://www.dropbox.com/1/oauth2/authorize?client_id=07o23gulcj8qi69&response_type=code&state=state
    2. Click "Allow" (you might have to log in first).
    3. Copy the authorization code.
    Enter the authorization code here:
    $ ./dbxcli ls <project folder on Dropbox>
    $ ./dbxcli get <project folder>/<date>_export.pkl.gz export.pkl.gz
    $ gzip -d export.pkl.gz
    ```

6. Switch to [DataCrunch web UI](https://cloud.datacrunch.io/dashboard/server-overview), start Jupyter notebooks, start developing

## At the end of each session

1. Save notebooks and other files, commit to version control; open Terminal from server Jupyter interface and do
    ```sh
    $ git add <files>
    $ git commit -m "<message>"
    $ git push
    ```
2. _(optional)_ If you developed a deep learning model and exported it to `export.pkl`, you will most likely want to keep the binary file out of git.
   For now, I'll upload it to Dropbox:
     ```sh
     $ gzip export.pkl
     $ curl -L https://github.com/dropbox/dbxcli/releases/download/v3.0.0/dbxcli-linux-amd64 -o dbxcli
     $ chmod a+x dbxcli
     $ ./dbxcli account
     $ ./dbxcli put export.pkl.gz <project folder>/$(date -I)_export.pkl.gz
     $ gzip -d export.pkl.gz
     ```
   Here we compressed the file, downloaded Dropbox command line client, authenticated with Dropbox (follow instructions after `./dbxcli account`)
   and uploaded the file, with a timestamp.
5. *Delete* DataCrunch server on the DataCrunch UI, in order to stop billing.


## Further notes
Data version control ([https://dvc.org/](https://dvc.org/)) seems quite a promising framework for keeping track of models and data.
I'll have to take a look at it some time later.
