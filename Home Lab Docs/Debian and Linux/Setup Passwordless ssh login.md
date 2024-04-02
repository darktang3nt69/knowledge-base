## Step 1: Generate a key pair

Use _ssh-keygen_ to generate a key pair consisting of a public key and a private key on the client computer. This command can be run on any modern Linux client distribution, the Terminal in macOS, or in the Command Prompt in Windows 10/11.

`ssh-keygen -t rsa`

The _-t rsa_ option specifies that the type of the key should be RSA. Other choices include DSA, ECDSA, and ED25519. Select the protocol your SSH connection will use.

When prompted, enter a filename for the key.

`Enter file in which to save the key.           (C:\Users\annem_000\.ssh\id_rsa):`

  
The default (_id_rsa_ in the _.ssh_ directory under the user’s home directory) works perfectly in most cases (and if this is to be your primary key, or only key, it is often the best option). Hit enter to accept the default (if you already have a key by this name, use whatever name you choose here throughout this tutorial in place of _id_rsa_). Then enter the passphrase when prompted.  
  

`Enter Passphrase (empty for no passphrase):`

  
Adding a passphrase is an important step for securing the local key, which otherwise will be usable by anyone who acquires the key itself. Choose a passphrase with the same rigor that you would use to create any secure passphrase. Some clients can be configured to save passphrases for a true “passwordless” access experience, while others may require it to be entered with each use. This will be covered in more detail later in the tutorial.

Type a passphrase (it will not be displayed, even though you are correctly entering it) and hit Enter (or hit Enter to continue with the default of no passphrase). Confirm the passphrase when prompted. The result will look similar to this:

![Command line screenshot of using ssh-keygen to generate a key pair](https://discover.strongdm.com/hs-fs/hubfs/ssh-passwordless-login-command-prompt.jpeg?width=586&name=ssh-passwordless-login-command-prompt.jpeg)

  
With the initial step to set up SSH passwordless login using _ssh keygen_ completed, you now have two files:

- _id_rsa_ contains the private key.
- _id_rsa.pub_ contains the public key.

## Step 2: Create SSH directory on server

Next, add the public key on the server you want to connect to. With your existing username and password, connect to the server using SSH, using whatever command line or client program you normally use for such connections. Check to see if the _.ssh_ directory already exists by attempting to list the files within it:

`ls .ssh`

If it does not, you will not be able to move into that directory and should instead create it:

`mkdir -p .ssh`

  
(Note the required dot at the beginning of the directory name, which makes this a hidden directory.)

## Step 3: Upload public key to remote server

### Uploading your public key with a Linux or macOS client

On a macOS or Linux client, use _ssh-copy-id_ to propagate the public key to the server, like this:

`ssh-copy-id user@somedomain`

Make sure to replace _user_ with a valid username from the server and _somedomain_ with the valid IP or domain of the server.

### Uploading your public key with a Windows client

With a Windows client, you can accomplish this task via the Windows Command Prompt. You will need to refer to the results of your earlier attempt to list the contents of the _.ssh_ directory and see if it contained a file called _authorized_keys_ or not.

A) **If you had to create the .ssh directory yourself, or if the remote server doesn’t already have an authorized_keys file**, on the client computer command line, enter the following to copy the public key to the _.ssh_ directory on the server (if you changed the name of your key from _id_rsa.pub_, change it here):

`scp .ssh/id_rsa.pub user@somedomain:~/.ssh/authorized_keys`

  
B) **If the remote server has an existing authorized_keys file**, the new key must be appended rather than overwriting the existing file. This is very important so that existing users do not lose access unintentionally. First, you will copy the file to the remote server. Then on the remote server, use the cat command to append it to the existing file:

On the client: _scp .ssh/id_rsa.pub user@somedomain:~/.ssh_

On the remote server: _cat .ssh/id_rsa.pub >> .ssh/authorized_keys_

On the remote server: _rm .ssh/id_rsa.pub_ (clean up after yourself and remove the now-unnecessary key file)

## Step 4: Test connection and configure an SSH agent

In your SSH session with the remote machine, update the permissions of the _.ssh_ directory and _authorized_keys_ file in case they need it:

`chmod 700 ~/.ssh`  
  
`chmod 600 ~/.ssh/authorized_keys`

  
Now, close your connection and your Terminal or Command Prompt. When you reopen it and try to connect to the remote server again from the client where you have your private key saved, you should receive a request to enter the passphrase instead of your username and password. Test it out:

`ssh user@somedomain`

  
👍 Success! Now, to avoid entering the SSH key passphrase every time:

1. You will need to use an SSH agent of some kind.  
      
    **For Windows:** You will use the OpenSSH Authentication Agent. The agent can be started by searching in the Windows Start menu for "Services," then double click on "OpenSSH Authentication Agent." Set the startup type to "Automatic" and click "Start."; Click Ok and Exit.  
      
    **For macOS and Linux:** The ssh-agent program already runs on session start for most Linux/Unix distributions. It provides an agent that you can add keys to and save passphrases. Once set up, the program will not require further interaction.  
      
    
2. At your command line prompt, in either case, type _ssh-add_. If you used the default _id_rsa_ naming for your key, that’s all you have to do. If you used a passphrase for the key, it will prompt you to enter it. Now the agent will remember your key and passphrase, and you won’t need to enter it on each use. You can also get more specific when adding keys (for example, if you used a different name for your key) with [ssh-add parameters](https://www.tutorialspoint.com/unix_commands/ssh-add.htm).

## Step 5: Back up SSH Keys

[A public key can be re-derived from a private key](https://askubuntu.com/questions/53553/how-do-i-retrieve-the-public-key-from-a-ssh-private-key#53555), but not vice-versa, making it especially important to back up private keys. To do so, simply back up the directory where they reside, which in our above examples, was the _.ssh_ directory in your user’s home directory. Both keys in the pair will be backed up because you generated them there, unless you removed one of them from the directory.