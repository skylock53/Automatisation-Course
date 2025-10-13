# Examination 1 - Understanding SSH and public key authentication

Connect to one of the virtual lab machines through SSH, i.e.

    $ ssh -i deploy_key -l deploy webserver

Study the `.ssh` folder in the home directory of the `deploy` user:

    $ ls -ld ~/.ssh

Look at the contents of the `~/.ssh` directory:

    $ ls -la ~/.ssh/

## QUESTION A

What are the permissions of the `~/.ssh` directory?

- The permissions of the `~/.ssh` directory are read, write and execute.

Why are the permissions set in such a way?

- Because of security reasons. The permissions are set in such a way so that only the user have the complete permissions to read, write and execute. If this was not the case, then any user that should not have the permissions would have access and could harm something. 

## QUESTION B

What does the file `~/.ssh/authorized_keys` contain?

- `~/.ssh/authorized_keys` contains public keys. These are used for login without a password.

## QUESTION C

When logged into one of the VMs, how can you connect to the
other VM without a password?

- You do this by generating a SSH-key, and then copy the public key to "Authorized Keys" for VM2. 

### Hints:

* man ssh-keygen(1)
* ssh-copy-id(1) or use a text editor

## BONUS QUESTION

Can you run a command on a remote host via SSH? How?

-Yes you can. You use "ssh deploy@dbserver "..."". The three dots stands for what command you want to run on the remote host via SSH. That could be "hostname" for example. 