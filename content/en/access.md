---
title: "Connect RCF resources"
---
The following information describes how users can access RCF resources. All users of RCF resources are responsible for knowing and complying with the RCF User Policy.
#Account credentials
To use the RCF provided resources you need to obtain an RCF user account. If you do not already have an RCF account, please refer to the Getting Started page for more information on obtaining an RCF account.

The RCF account uses your name, surname or registration number for the username and password usually used for the IT services of the University of Naples "Parthenope".

**Important**: RCF does not store your password and we are unable to reset your password. If you need assistance with the password, consult the University web page.

# Connect via SSH
Secure Shell (SSH) is a protocol that provides secure command line access to remote resources such as purpleJeans. Using SSH, you can remotely log into your purpleJeans account and interact with the purpleJeans high-performance compute cluster.

## Access via Macintosh and Linux
Most Unix-like Operating Systems (Mac OS X, Linux, etc.) provide an ssh utility by default which can be accessed by typing the ssh command in a terminal window.

To access purpleJeans from a Linux or Mac computer, open a terminal and from the command line enter:

```sh
$ ssh <name.surname>@purplejeans.uniparthenope.it
```

Or

```sh
$ ssh <matricola>@purplejeans.uniparthenope.it
```

To enable X11 forwarding when connecting to purpleJeans with ssh, you need to include the -Y flag.

**Important**: Macs must have the XQuarz software installed.

# Connect using Windows and SSH
Windows users running the April 2018 release of Windows 10 will have SSH enabled from Powershell by default. All other Windows users will first need to download an ssh client to interact with the Unix remote command line. We recommend MobaXterm, client, although other options are available. Once you have installed the MobaXterm client on your local computer, open the MobaXterm client (link: https://mobaxterm.mobatek.net) and click on the Sessions icon in the upper left corner of the client. Then perform the following numbered steps, shown in the following figure, to establish a connection to purpleJeans:

- Click the SSH tab to expand the SSH login options.
- In the Remote Host field input: purpleJeans.uniparthenope.it (to connect to the purpleJeans cluster)
- Select the Specify Username button and enter your uniparthenope username.
- Proceed with login by clicking the OK button.

# Connect via XFast
The software for ubiquitous access via a dedicated client or web browser is being acquired).