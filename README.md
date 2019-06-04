# Hashicorp Vault setup using Ansible Role

We are going to create an Ansile Role for Vault setup so we can reuse it. We will begin by creating a new user account named "vault" which will help with a secure setup. We will use this account to isolate the ownership of vault. We don't create any home directory or shell for this user so that user can't log in to a server.

Next, we need to download vault archive from here on our remote vault instance. This will give a zip archive file. To unzip vault archive, we need to install unzip so we can unzip vault archive and takeout needed binary. Once this is done, we need to unzip vault archive, move our vault binary to "/usr/local/bin" and make vault user as the owner of this binary with reading and execute permissions.

We need to set binary capabilities on Linux, to give the Vault executable the ability to use the mlock syscall without running the process as root.

We need to setup systemd init file to manage the persistent vault daemon. We need to set below content into systemd service file. Finally, start the vault server.

### Setup

Some changes are required in these files in order to make it run:
* inventory
  Remove the default ip and provide server IP address of the instance on which we want to run this playbook.
* ansible.cfg
  Especify the name of remote_user you are using, it changes depending on the version of linux and provider (ec2-user, ubuntu, etc)
  Add the private_key_file location in your instance (place it in the same directory as  the inventory and ansible.cfg to prevent errors)
* vault.hcl.j2 (the one in: roles/vault/templates/)
  As we are using S3 as Vault backend, please provide access_key, secret_key, bucket and region (write everything between quotation marks).

Once done run command:

ansible-playbook playbook.yml

## Troubleshooting

Before further reading check typo inside edited files where changes where made, after that maybe this could help:

_Error initializing: Put http://0.0.0.0:8200/v1/sys/init: dial tcp 0.0.0.0:8200: connect: connection refused_
  check enviroment vars and make sure that your playbook´s locations is algo in $PATH also make sure you added VAULT_ADDR address properly.
  make sure both inventory and vault.hcl.j2 list "http" instead of "https" if one of those isn´t "http" isn´t going to work.
  delete the info inside the bucket, sometimes it requires an empty bucket to work.
_/bin/sh: vault: command not found"_
  vault isn´t properly initialized, run the playbook again.
  make sure your instance works with the folder path listed in the playbook, if no, change the folder path or create it inside your           instance.
_vault is already initialized_
  vault is already running in the machine, if the playbook isn´t finished by this point, you have to kill the instance and try it again;     vault service once initialized, shares it´s keys and NEVES does it again.
  vault began it´s service, but wasn´t able to promt it´s keys, delete the instance and try it again, make shure, there is no security       group or ACL blocking the conection to port 8200.
  
