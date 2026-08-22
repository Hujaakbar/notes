# Authentication With GitHub Using SSH Keys

GitHub does not support signing-in using username and password anymore. As an alternative, it is possible to use SSH.
When you connect via SSH, you authenticate using a private key file on your local machine. In simple words,
instead of using a password, this method  uses a long cryptographic string known as  private key.

It is also possible to use an SSH key to sign commits. 

To authenticate with GitHub using ssh keys, we need to 

1. generate ssh keys
1. register it with GitHub

## Genrating SSH Keys

1. Before generating new ssh keys, check whether ssh keys already exist

  ```bash
  ls -al ~/.ssh
  ```

1. Generate SSH Keys
  If ssh keys do not already exist, then create new ones using following commands:

  ```bash
  ssh-keygen -t ed25519 -C "my-email-address@xyz.com"
  ```

  -  `-t` option specifies the type of key to create. `ed25519` is the default.
  - `-C` option provides a new commnet

  The above command creates a private key file (`~/.ssh/id_ssh-key-type`) and public key file (`~/ssh/id_ssh-key-type.pub`).

1. Register Public SSH Key To GitHub for Authentication and Commit Signing

1. Check the SSH connection

  ```bash
  ssh -T git@github.com
  ```

1. When cloning repository or adding the remote use ssh 

   ```bash
   git clone git@github.com:Username/repotisotry_name.git
   
   git remote add origin git@github.com:Username/repository_name.git
   ```




