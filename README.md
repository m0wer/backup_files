# Backup Files

Ansible playbook for backing up files from remote hosts into a git repository.

Every time it's run, it will copy a list of files from each of the remote hosts
and commit the changes (if any) to a local git repository.

Useful for keeping the versions (including the most recent one) of configuration
files that are edited by hand on remote hosts.

The backed up files will be stored in the local git repository keeping their
path in a folder named as the remote host's. For example, the file
*/etc/hostname* from the host *rpi1* will be stored in the local git repository
at *rp1/etc/hostname*.

## How it works

This playbook will:

1. Check for changes and copy the selected backup files that changed from the
   remote hosts to the local git folder.
1. Commit the changes of the copied files if any of them changed.

   This is done per host. The commit message will look like:

   ```
   Config snapshot of rpi1 taken at 2020-08-25 19:13:12.
   ```

1. Run `git push` from the local machine on the local git folder.

## Requirements

* Ansible (both in the local machine and the remote one(s)).

## Setup

1. Clone this repository.
1. Setup a local git repository, for example by running:

   ```bash
   mkdir /home/user/conf_backup
   cd /home/user/conf_backup
   git init
   git config user.email "[email]"
   git config user.name "[username]"
   touch README.md
   git add README.md
   git commit -m "Initial commit."
   ```

   Optionally, add a remote upstream to this repo.

1. Copy and edit the playbook variables from *group_vars/all.yml* to your
   inventory file, which you can place under *custom/* since its gitignored.

   An example inventory (*custom/home_servers/hosts.yml*) could look like:

   ```yaml
   ---

   home_servers:
     hosts:
       rpi1:
         backup_files:
            - /etc/hostname
            - /etc/hosts
       rpi2:
         backup_files:
            - /etc/hostname
     vars:
         ansible_python_interpreter: /usr/bin/python3
         local_git_repo_path: /home/user/conf_backup
   ```

   **Note**: The hosts must be present in your *~/.ssh/config* file, that could
   look like:

   ```
   Host rpi1
      HostName 192.168.1.10
      Port 22
      User pi
   Host rpi2
      HostName 192.168.1.20
      Port 22
      User pi
   ```

1. From the repository's base directory run:

   ```bash
   ansible-playbook -i [inventory] main.yml --ask-become-pass
   ```

   Alternatively to `--ask-become-pass`, the `sudo` password can be stored in
   the inventory file in the variable `ansible_become_pass` in plain text,
   using [vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
   or as a command to get it from a password manager (e.g. `"{{ lookup('pipe',
   '[command that returns the password]') }}"`).

## Playbook variables

* `backup_files`: List of the paths of the files to backup. For example:

   ```yaml
   backup_files:
     - /etc/hostname
     - /etc/hosts
   ```

* `fetch_fail_on_missing`: (Default: `true`) Boolen value on whether the
  `fetch` module should fail it the remote file doesn't exist or not.

  Setting it to `false` might be convenient if you want to set the
  `backup_files` for a gorup of the hosts but not all of them contain all of
  the files.
* `git_push_changes`: (Default: `false`) Boolean value, whether to push changes
  from the local git repository to a remote one or not. For example: `true`.
* `local_git_repo_path`: Local path for the git repository where the remote
  files will be saved. For example: `/home/user/conf_backup`.

## Author

* m0wer: m0wer (at) autistici (dot) org
