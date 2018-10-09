Ansible Role: Backup
====================

[![Build Status](https://travis-ci.com/gantsign/ansible_role_backup.svg?branch=master)](https://travis-ci.com/gantsign/ansible_role_backup)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-gantsign.backup-blue.svg)](https://galaxy.ansible.com/gantsign/backup)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/gantsign/ansible_role_backup/master/LICENSE)

Role to backup and restore files and directories. Uses
[rsync](https://rsync.samba.org/) but intended to be used with a locally
mounted backup drive.

During provisioning this role will restore any previously backed-up files and
directories. Once restarted the OS will perform incremental backups every 5
minutes and during shutdown.

The backup is a simple mirror of the source directory, previous file versions
are discarded. The main purpose is to persist files between rebuilds of your
local development VM. If you can't afford to lose/replace your files, use a
remote backup solution with versioning instead.

This role is primarily indented for backing-up and restoring the contents of a
users home directory.

Requirements
------------

* Ansible >= 2.5

* Linux Distribution

    * Debian Family

        * Ubuntu

            * Bionic (18.04)

    * Note: other versions are likely to work but have not been tested.

Role Variables
--------------

The following variables will change the behavior of this role:

```yaml
# The frequency to perform the backup
backup_frequency_minutes: 5

# User who owns the files to backup/restore
backup_user: # Required

# The source directory containing the files to backup (must end with a /)
backup_src:  # Required

# The destination directory to backup the files to (must end with a /)
backup_dest: # Required

# The rsync filter of files/directories to include/exclude
backup_filter: |
  !
  # Include everything
  + /*

# Directory for backup scripts
backup_script_dir: '~/.backup'

# The name to use for cron
backup_cron_name: backup

# The name to use for systemd service
backup_service_name: backup
```

Example Playbooks
-----------------

Here's a simple example playbook:


```yaml
- hosts: servers
  roles:
    - role: gantsign.backup
      backup_user: example_username
      backup_src: /home/example_username/
      backup_dest: /mnt/backup/example_username/
      backup_filter: |
        # Reset filter
        !
        # Include file / directory
        + /include_me
        # Include subdirectory (only needed of you're going to exclude other sub-directories)
        + /include_me/me_as_well
        # Exclude everything but me_as_well
        - /include_me/*
        # Exclude by file/directory name
        - tmp
        # Exclude everything else
        - /*
```

Here's a real world example playbook:

```yaml
- hosts: servers
  roles:
    - role: gantsign.backup
      backup_user: vagrant
      backup_src: /home/vagrant/
      backup_dest: /var/persistent/home/vagrant/
      backup_filter: |
        !
        + /.atom
        + /.atom
        + /.atom/config.cson
        - /.atom/*
        + /.bash_history
        + /.config
        + /.config/Code
        + /.config/Code/User
        + /.config/Code/User/settings.json
        - /.config/Code/User/*
        - /.config/Code/*
        - /.config/*
        + /.gitconfig
        + /.gnupg
        + /.m2
        - /.m2/repository
        - /.m2/wrapper
        + /.ssh
        - /.ssh/authorized_keys
        + /workspace
        + /.zsh_history
        - target/*
        - build/*
        - node_modules
        - /*
```


More Roles From GantSign
------------------------

You can find more roles from GantSign on
[Ansible Galaxy](https://galaxy.ansible.com/gantsign).

Development & Testing
---------------------

This project uses [Molecule](http://molecule.readthedocs.io/) to aid in the
development and testing; the role is unit tested using
[Testinfra](http://testinfra.readthedocs.io/) and
[pytest](http://docs.pytest.org/).

To develop or test you'll need to have installed the following:

* Linux (e.g. [Ubuntu](http://www.ubuntu.com/))
* [Docker](https://www.docker.com/)
* [Python](https://www.python.org/) (including python-pip)
* [Ansible](https://www.ansible.com/)
* [Molecule](http://molecule.readthedocs.io/)

Because the above can be tricky to install, this project includes
[Molecule Wrapper](https://github.com/gantsign/molecule-wrapper). Molecule
Wrapper is a shell script that installs Molecule and it's dependencies (apart
from Linux) and then executes Molecule with the command you pass it.

To test this role using Molecule Wrapper run the following command from the
project root:

```bash
./moleculew test
```

Note: some of the dependencies need `sudo` permission to install.

License
-------

MIT

Author Information
------------------

John Freeman

GantSign Ltd.
Company No. 06109112 (registered in England)
