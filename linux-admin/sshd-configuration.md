# Configuring SSHD and SSH Keys

## Overview
This guide will walk you through setting up a SSH server, basic SSH security, and how to create and use SSH keys. You will need a server with a public IP address and this guide assumes you will be using a Debian-based Linux operating system.

## Audience
This guide is primarily for system adminstrators who want secure shell access to their servers. Profiency with the command line is needed.

## Prerequisites
Before starting, you should have:

 - A Debian-based Linux system (Debian 11/12 or compatible)
 - Root or sudo access
 - An active internet connection

## Step 1: Update the System
A good practice before installing new software on a Debian system is to update your software repositories and system software:

```
sudo apt update
sudo apt full-upgrade
```

Reboot the system if required.

## Step 2: Install SSHD
You can find the SSH server software in the Debian repository:

```
sudo apt install openssh-server
```

Once complete, you can verify that the server is active and running with the following:

```
sudo systemctl status sshd
```

# Step 3: Basic SSHD Security Configuration
Open the SSHD configuration file for editing:

```
sudo nano /etc/ssh/sshd_config
```

The important parts of the file to modify are listed below:

```
Port <non-default number>
Protocol 2
PermitRootLogin no
MaxAuthTries 3
AllowUsers <usernames>
PermitEmptyPasswords no
```

Here are some explanations for the configuration options:

 - **Port**: By default, `sshd` runs on port 22. Many automated hacking attempts are based around that assumption, so by running on a different port number you can avoid many attempts to access your system.
 - **Protocol**: Newer versions of `sshd` should not accept older protocols, but we can manually define using the newer one here.
 - **PermitRootLogin**: Disallow logging into the server as root. This forces attackers to know both a username and password as well as the root password.
 - **MaxAuthTries**: The maximum number of attemptsa user may make before the connection is terminated.
 - **AllowUsers**: Only these specified users will be allowed to log into the system with SSH.
 - **PermitEmptyPasswords**: Disallow logging into accounts with NULL passwords.

Once complete, save and exit the editor, then reload the SSHD server:

```
sudo systemctl reload sshd
```

At this point, you will have a functioning and secure SSH server.

# Step 4: SSH Key Generation
Optionally, we can use SSH keys to log into our system. Using these keys, you can have a higher level of security for your server by disallowing password logins.  Only users with verified and allowed keys will be allowed access to the server.

First, generate your SSH keys on the system that you want to login from:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Follow the prompts and save the key to the default location. You may use a passphrase for added security, in case your private key is ever stolen.

To prevent having to type the password in everytime, and save it for the current session, add the key to the SSH Agent:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

You may add this to your .bash_profile (or similar) to do this on every login.

Now copy that key to the SSH server:

```
sudo ssh-copy-id -i ~/.ssh/id_ed25519.pub username@example.com
```

You may add the `-p <port number>` flag if you are running on a different port.

Repeat this process for every computer you would like to login to your server from.

## Configure SSHD to Disallow Password Logins

## Troubleshooting

## Conclusion