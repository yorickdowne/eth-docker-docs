---
title: "Running Eth Docker on a service account"
sidebar_position: 11
sidebar_label: Eth Docker with multiple users
---

## Using a "service account"

You may want to run Eth Docker on one user, and then have other users be able to administer it. For this
example, the user that installs Eth Docker will be `node`, the two admin users will be `alice` and `bob`,
and all three belong to the `node-admin` group.

`alice` can `sudo`

As `alice`, create the `node` user and `node-admin` group
- `sudo adduser node`
- `sudo addgroup node-admin`
- `sudo adduser node node-admin`
- `sudo adduser alice node-admin`
- `sudo adduser bob node-admin`

Keep `node` from logging in via ssh
- `sudo nano /etc/ssh/ssh_config.d/99-disable-node-login.conf`
```
DenyUsers node
```
- Save and close
- `sudo systemctl restart ssh`

Become `node` and download Eth Docker, and set permissions
- `sudo su - node`
- `cd ~ && git clone https://github.com/ethstaker/eth-docker.git`
- `chown node:node-admins .`
- `chown -R node:node-admins ./eth-docker`
- `find ./eth-docker -type d -exec chmod g+s {} +`
- `exit`

As `alice` again, install prerequisites and configure Eth Docker
- `/home/node/eth-docker/ethd install`
- Consider saying "yes" to being able to call `ethd` from anywhere
- `source ~/.profile`
- `ethd config`
- OR if you opted out of calling `ethd` from anywhere
- `/home/node/eth-docker/ethd config`

Make `bob` part of the `docker` group, unless they have `sudo` rights and you prefer that for them
- `sudo adduser bob docker`

If you opted in to being able to call `ethd` from anywhere, tell `bob` to add this to their `~/.profile`
```
alias ethd=/home/node/eth-docker/ethd
cat /home/node/eth-docker/.motd
```

The `node` user now owns Eth Docker, but cannot run `docker` commands itself. If you want someone to be able
to become `node` and run `docker` commands, add `node` to the `docker` group as well: `sudo adduser node docker`

If `bob` does not have `sudo` rights themselves, they'll be able to administer Eth Docker, but won't be able
to run commands that make system level changes, such as `ethd install`. `alice` has full capabilities, as they
can `sudo`.
