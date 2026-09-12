# SSH

SSH (Secure Shell) is a network protocol that provides a secure way to access and manage remote systems over an unsecured network.

```bash
# Basic syntax to initiate an SSH connection
ssh [options] [user@]host
```

For more information see:

- [OpenSSH full guide](https://www.youtube.com/watch?v=YS5Zh7KExvE&t=2203s&pp=ygUQc3NoIGxlYXJubGludXh0dg%3D%3D)
- [Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)
- [SSH Keys](https://wiki.archlinux.org/title/SSH_keys)


## Install

To use SSH, you need an SSH client installed on your local host and an SSH server running on the remote host.

On Archlinux, when you install the [OpenSSH](https://wiki.archlinux.org/title/OpenSSH) package you get both (plus an [SSH agent](https://wiki.archlinux.org/title/SSH_keys#SSH_agents)) so the same machine can act as a client or as a server. On other distributions, like Ubuntu, you may need to install different packages.

```bash
# Install OpenSSH
sudo pacman -S openssh

# Enable and start the service
sudo systemctl enable sshd
sudo systemctl start sshd
```


## Configuration files

Global configuration files:

- `/etc/ssh/sshd_config` - global server settings (see changes below)
- `/etc/ssh/ssh_config` - global client settings (everything is commented by default)

User configuration files:

- `~/.ssh/config` - user client settings (overwrite global settings)
- `~/.ssh/authorized_keys` - contains the public keys of all hosts that are authorized to connect to this machine
- `~/.ssh/known_hosts` - contains the IP addresses and public keys of all hosts this machine has connected to.


## SSH Keys

While you can use the typical *user/password* approach to connect to a remote machine via SSH, it is hardly encouraged that you use SSH keys instead.

SSH keys are always generated in pairs with one known as the private key and the other as the public key. Both keys are stored in the `~/.ssh/` directory.

The private key is known only to the client machine and it should NEVER be shared.

The public key can be shared freely with any SSH server you wish to connect to.

Only the private key can decrypt a message encoded with the public key. Any machine in possession of the private key will be allowed to connect to any other machine that has its public key counterpart registered in the `~/.ssh/authorized_keys` file.


## Client setup

#### Generate a new SSH key pair

> Do not reuse keys, each server/service should have a different key. That way if one key is compromised all other servers/services remain safe.

```bash
ssh-keygen -t ed25519
```

The key type `ed25519` relies on elliptic curve cryptography, which provides the same level of security with smaller keys. Smaller keys result in less computationally intensive operations (faster key creation, encryption and decryption and reduced storage and transmission requirements).

Do not use a comment, the default is `{USER}@{HOSTNAME}`. The comment is appended to the content of the public key, since the content of the public key is copied to the server's `~/.ssh/authorized_keys` file, you can easily identify what machines have access to a server by inspecting that file on the server.

Use the remote hostname as the name of the key. This will help you identify the purpose of each key in the `~/.ssh` directory (be careful not to use the name of any pre-existing key, it will overwrite the key without any warning).

Always add a passphrase. Without the passphrase the key is unusable, this will buy you time to replace the keys if they ever get leaked. Furthermore, without a passphrase, you must trust the root user, as they can bypass file permissions and will be able to access your unencrypted private key file at any time.
  
If you ever need to change the passphrase, use `ssh-keygen -f ~/.ssh/<key> -p`, there is no need to generate a new key.
  
By using an SSH-agent we only need to type the passphrase once (see below).

#### Instruct the SSH client how to connect to each server

By including the connection details for each server in the SSH client configuration file we greatly simplify the process of connecting to those servers.

The `~/.ssh/config` file should look something like this:

```bash
# Instruct the SSH-agent to automatically add the keys for all connections
AddKeysToAgent yes

# Server 1
Host {SERVER_ALIAS}
    Hostname {SERVER_IP}
    Port {PORT} # Optional, defaults to 22
    User {SERVER_USER} # Optional, in case the client and server use different users
    IdentityFile "~/.ssh/{PRIVATE_KEY}"

# Server 2
Host {SERVER_ALIAS}
    Hostname {SERVER_IP}
    IdentityFile "~/.ssh/{PRIVATE_KEY}"

# Server 3
Host *github.com
    IdentityFile "~/.ssh/github"

# Etc...
```

#### Copy the public key to the server

```bash
ssh-copy-id -i ~/.ssh/{PUBLIC_KEY}.pub [{SERVER_USER}]@{SERVER_IP}
```

The `ssh-copy-id` command will copy the content of the given public key and append it to the given user's `~/.ssh/authorized_keys` file on the given server (you will be prompted for the user's password).

> Apparently root password authentication is disabled by default. If you receive an error when trying to copy the ssh key using the root user you must explicitly change the values of `PermitRootLogin` and `PasswordAuthentication` to `yes` and restart the `sshd` service. Don't forget to revert the change on the server after the SSH key is copied.

#### Test the connection

```bash
ssh {SERVER_ALIAS}
```

If everything was done correctly you should now be logged into the given server.

### SSH agent

We can use the SSH agent included in OpenSSH to avoid having to type the passphrase of each key every time we use it. The role of the SSH agent is to save the passphrase for us so that we only need to type it once.

For Xorg, the SSH-agent is automatically started [as a wrapper program](https://wiki.archlinux.org/title/SSH_keys#ssh-agent_as_a_wrapper_program) by the shell alias `startx`. With this approach exactly one instance of the SSH agent will live and die for the entire X session and it works across all terminals.

For Wayland, see [start the ssh-agent with systemd user](https://wiki.archlinux.org/title/SSH_keys#Start_ssh-agent_with_systemd_user).


## Server setup

> Before applying the following changes make sure you can connect to this host (from any other host) using SSH keys.

#### Disable password authentication

If your network configuration exposes your machines to the internet you MUST disable password authentication, specially for the root user. Hackers can easily bypass the password security measure using a brute force attack.

Open the SSH server configuration file:

```bash
sudoedit /etc/ssh/sshd_config
```

The following settings will disable password authentication for all users, allowing connections only via SSH keys. It does not only improve security, but the user experience as well.

```bash
# disable password authentication for root (super important!)
PermitRootLogin prohibit-password
# disable password authentication for all users (optional)
PasswordAuthentication no
```

#### Restart the SSH daemon to apply the changes

```bash
systemctl restart sshd
```

Now only users with previously authorized keys can access this machine via SSH.


## Debuggin

If a connection fails do the following.

On the client, the `-v` option (`ssh -v ...`) will give you a verbose output of the connection details, there you can see what failed.

On the server, `systemctl -fu sshd` will let you watch the log file entries for the `sshd` service in real time