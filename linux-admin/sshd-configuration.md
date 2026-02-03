# Configuring sshd and SSH Keys

## Overview
This guide will walk you through setting up an SSH server, basic SSH security, and how to create and use SSH keys. 

The default configuration values ease of use and accessibility over security, and this guide will help you lock down your sshd server. Securing your SSH server is vital as it is one of the most common entry points on Linux systems, with many automated scanning and brute force attacks made to work against known vulnerabilities being ran to target any SSH server found on the internet.

This guide covers basic SSH hardening and is not a complete server security checklist.

You will need a server with a public IP address and this guide assumes you will be using a Debian-based Linux operating system.

## Audience
This guide is primarily for system administrators who want secure shell access to their servers. Proficiency with the command line is needed.

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

## Step 2: Install sshd
You can find the SSH server software in the Debian repository:

```
sudo apt install openssh-server
```

Once complete, you can verify that the server is active and running with the following:

```
sudo systemctl status sshd
```

## Step 3: Basic sshd Security Configuration
Open the sshd configuration file for editing:

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
AllowGroups <group names>
PermitEmptyPasswords no
```

Here are some explanations for the configuration options:

 - **Port**: By default, `sshd` runs on port 22. Many automated hacking attempts are based around that assumption, so by running on a different port number you can avoid many of these automated attempts to access your system. This is, however, "security through obscurity" and does not replace proper authentication controls.
 - **Protocol**: Setting this to "2" is redundant, as newer versions of OpenSSH ignores Protocol 1 completely, but we can manually define the newer version here for clarity.
 - **PermitRootLogin**: Disallow logging into the server as root. This eliminates automated attempts to gain access with a username present on all Linux systems.
 - **MaxAuthTries**: The maximum number of attempts a user may make before the connection is terminated.
 - **AllowUsers**: Only these specified users will be allowed to log into the system with SSH. If you have multiple users on the system, if may be preferable to use the **AllowGroups** directive to allow certain groups as opposed to specific users.
 - **PermitEmptyPasswords**: Disallow logging into accounts with NULL passwords.

Once complete, save and exit the editor, then reload the sshd server:

```
sudo systemctl reload sshd
```

At this point, you will have a functioning and secure SSH server.

## Step 4: SSH Key Generation
Optionally, we can use SSH keys to log into our system. Using these keys, you can have a higher level of security for your server by disallowing password logins.  Only users with verified and allowed keys will be allowed access to the server.

First, generate your SSH keys on the system that you want to login from:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Follow the prompts and save the key to the default location. You may use a passphrase for added security, in case your private key is ever stolen.

### Using SSH Agent
If you protected your private key with a passphrase, you can use an SSH agent to avoid re-entering the passphrase for every connection during a session.

Start the SSH agent and add your private key:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

You will be prompted for your key’s passphrase once. The decrypted key will remain available for the duration of the session.

On many desktop Linux environments, an SSH agent may already be started automatically when you log in. Look for a dialog box asking for a passphrase. On servers or minimal environments, however, you may need to start the agent manually.

If desired, you can configure your shell to start the SSH agent automatically on login. The exact configuration file varies by system and shell (for example, .profile, .bash_profile, or .bashrc). Care should be taken to avoid starting multiple agents unintentionally.

### Copying Public Keys
Now that we have generated a public key, we can copy that key to the SSH server. Issue the following command from the remote host:

```
sudo ssh-copy-id -i ~/.ssh/id_ed25519.pub username@example.com
```

You may add the `-p <port number>` flag if you are running on a different port.

Repeat this process for every computer you would like to login to your server from.

### Configure sshd to Disallow Password Logins
**DO NOT LOCK YOURSELF OUT.** Be sure to test and verify connections from your allowed hosts to make sure you can connect to your SSH server before proceeding. Keep your active session open until you verify that you can access your server after reloading.

Open the sshd configuration file for editing:

```
sudo nano /etc/ssh/sshd_config
```

Modify (or add) the following value:

```
PasswordAuthentication no
```

Reload the sshd server:

```
sudo systemctl reload sshd
```

Your server will no longer accept password-based logins and will only accept connections from machines with trusted keys.

## Troubleshooting
**Firewall**: Be sure that the configured port is allowed through your firewall.

**-p vs -P**: For port specification, some the `ssh` commands use `-p` (lowercase) while others command such as `scp` use `-P` (uppercase). 
