Welcome to Autops awx-migrate
=============================


**awx-migrate** is a command line tool for Ansible AWX. It leverages the
tower-cli command, makes a full backup of an AWX instance, and adds the secrets
to the exported credentials, which tower-cli leaves empty.

It then dumps the whole export including credential secrets in json to stdout,
which you can redirect to a file. It also takes al config settings from the
database - including LDAP settings and adds or updates them in a new instance.

This tool allows you to fully migrate an AWX setup, and is a workaround for not
being able to upgrade AWX.
(See https://github.com/ansible/awx/blob/devel/DATA_MIGRATION.md)

For more information, on tower-cli look at http://tower-cli.readthedocs.io.


Things that are not migrated
----------------------------------

* logs
* encrypted data in the conf_setting table (e.g. LDAP Bind password)


Requirements
============

Python
------

* Python 2.7
* tower_cli
* psycopg2
* cryptography


Configuration
-------------

AWX - separately configure how to connect to both the old and the new instance

* The AWX secret_key, used to encrypt and decrypt secrets
* credentials for an AWX admin user
* connection information and credentials to directly access the postgresql DB

and this for both the source and destination AWX instance and its postgresql database


How-to
=====

* edit `awx-migrate-wrapper` and set the right `AWX_SRC_*` and `AWX_DST_*`
  environment variables as needed, the former being for the source instance,
  the latter for the destination instance
* configure `tower-cli config` to connect to the *source* awx instance you want
  to migrate from
* execute `awx-migrate-wrapper` and redirect stdout to a temp file,
  say `awx-data.json`
* watch stderr for possible error messages
* if you use LDAP or other config settings that require an encrypted secret,
  update it manually
* configure `tower-cli config` to connect to the *destination* awx instance you want
  to migrate to
* restore awx data with `tower-cli send awx-data.json`


Bugs
====

* Tested migrating from AWX 1.0.7.2 to 2.0.1.
  Several objects where not restored due to missing users: there's bug in 1.0.x?
  where the export of users is broken/missing so that tower-cli cannot import
  them again.
  (see https://github.com/ansible/tower-cli/pull/586#issuecomment-432176155)
* Currently this script assumes all credentials have a unique name, which
  however isn't enforced by AWX. If you have those, PR is welcome!
* Encrypted secrets in the conf_settings table are not decrypted. Need info on
  how to decrypt/encrypt them


Warning
=======

This script was written for and tested in a specific setup and particular
environment, lot's of issues or bugs can still arise
