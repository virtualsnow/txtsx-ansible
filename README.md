# Ansible to construct txt.sx infrastructure

Duplicates all of the infrastructure at `txt.sx` with Ansible and Docker. More to come.

### Roles
- **directory**:  static lander served by nginx, generated from Jinja2 templates. ([txt.sx](https://txt.sx))
- **readme**: static blog served by nginx, generated by Pelican, using custom themes ([readme.txt.sx](https://readme.txt.sx))
- **paste**:  temporary self-destructing paste & file server built on [linx-server](https://github.com/andreimarcu/linx-server) with custom themes. ([paste.txt.sx](https://paste.txt.sx))
- [ezxss](https://github.com/ssl/ezXSS): blind-xss analysis framework like xsshunter. ([x.txt.sx](https://x.txt.sx))
- **haproxy**:  sets up haproxy for ssl termination and forwards traffic to corresponding container.
- **docker**: initializes docker for user on host.
- **common**: some basic setup.


### Setup
Expects inventory servers to have an `ansible` user, with a password stored in the vault as `ansible_sudo_pass`. This user must be added to sudoers so they can operate Docker.

Example setup using the same ssh configuration as `root`:
- `adduser ansible`
- `password ansible`
- `cp -r /root/.ssh /home/ansible/.ssh`
- `chown -R ansible:ansible /home/ansible/.ssh`
- `usermod -a -G sudo ansible`
- `apt-get update && apt-get upgrade`

Certificates need to be manually added at this time.  Put your certificates at `/etc/ssl/private/` and change the domains for the `haproxy` task to yours.

If you're pulling from __private repositories__, please see the appropriate section below for more setup requirements.


### Running
- Checking the playbook for errors: `ansible-playbook deploy.yml --syntax-check`
- Copy `staging` inventory file to `production` and edit the contents to fit your architecture.
	- `ansible-playbook deploy.yml` Runs with local ansible.cfg and default inventory file.
	- `ansible-playbook deploy.yml --vault-password-file .vault_pass` Supplies vault password from file so you are not prompted.
	- `ansible-playbook deploy.yml -i production--vault-password-file .vault_pass` Runs deployment against the local inventory file named `production`
	- `ansible-playbook deploy.yml --vault-password-file .vault_pass --start-at-task "full name of task"` Skips steps, useful if Ansible was interrupted or there was an error.


If you only want to deploy a subset of the systems, simply don't list any hosts under the unwanted categories in your inventory file.


You can customize the services by their configuration files, usually in `/roles/rolename/files` or the role variables `/roles/rolename/vars`.


### Vault-tech:
- Create a new password vault: `ansible-vault create filename`
- Edit an existing vault: `ansible-vault edit filename`
- Edit an existing vault using a password file: `ansible-vault edit vault --vault-password-file .vault_pass`
- For one vault, polymorph vars, change devvault identifier.
- `ansible-vault create --vault-id devvault@prompt vaultfile.yml`
- In tasks, `include_vars: vault` or any other vaultfile, to access vault contents.
- Use double curly brace to reference vaultfile variables. `"{{ somesecret }}"`
- Run:
	- `ansible-playbook deploy.yml --vault-password-file .vault_pass`
	- `ansible-playbook deploy.yml --vault-password-file .vault_pass --start-at-task "sometask"`

### Pulling from private Git repositories (Github):

If your blog is on a private git repository, Ansible may have trouble accessing it.  You may be able to add a low authority user that ansible can use for deployment if you control the git service.

However, if you don't control the service, such as Github, it may be best to use SSH agent forwarding.  This allows your computer to authorize to Github on your behalf, while the script runs.

**Do not use SSH agent forwarding to connect to hosts you do not trust.**

To integrate SSH agent forwarding to Ansible:
- Make sure ssh-agent is running. `eval \`ssh-agent\``.
- Make sure your key is added to ssh-agent `ssh-add path/to/priv_rsa`
- Check the key is added: `ssh-add -l`
- Modify `.ssh/config` to include a Host entry for the inventory entries with `ForwardAgent yes`.  Specify your `IdentityFile` and set `IdentitiesOnly`  as well so all your keys aren't sent, only the relevant ones.
- In your `ansible.cfg`, which can be system-wide or project-specific, include `ssh_args=-o ForwardAgent=yes` under the `[ssh_connection]` section.
- Make sure `accept_hostkey: yes` is set for ansible git task, and that you're using the `ssh` git endpoint instead of `https`. It should look like `git@github.com:yourname/yourrepo.git`.


### todo
- restart haproxy on signal change
- auto initialize ansible user under sudoers with password from vault (make setup easy)
- fix/automate certificate initialization via letsencrypt.
- mysql image initialization bug
- pull all txt.sx specific content to vars.

