pulp_database_config
====================

Configure the database for Pulp 3

More specifically, this role does the following via `django-admin`:

1. Create and run migrations. (modifies the database schema for Pulp)
2. Set the Pulp admin user's password.

This role depends on the [pulp_common](../../roles/pulp_common) role, see it for many more variables on configuring pulp_database_config.

Role Variables
--------------

* `pulp_default_admin_password`: Initial password for the Pulp admin. Only affects Pulp
  during initial install, not upgrades/updates or re-running the installer for any other
  reason. **Required**.
* `pulp_db_fields_key`: Relative or absolute path to the Fernet symmetric encryption key
   one wants to import. The path is on the Ansible management node.
   It is used to encrypt certain fields in the database (such as credentials.)
   If not specified, a new key will be generated. (Only generated if one doesn't exist.)

Role Variables for advanced usage
---------------------------------

* `pulp_database_config_host`: pulp_database_config is designed to be run against only 1
  host. In the event that it is accidentally run against multiple hosts, this is the only
  host that will run pulp_database_config's tasks that actually modify the state of the
  host/application. Its database fields encryption key will be copied to all the other
  hosts in later roles. If not specified, a host is randomly picked from suitable hosts
  (hosts that already have the database fields encryption key.) If specified, it must
  match the host's name in the Ansible inventory exactly.
* `pulp_force_change_admin_password`: Whether to always change the password to that which
  is specified in `pulp_default_admin_password`. Defaults to `false`.
  Note: If set to true, the installer will not be idempotent.

Shared Variables
----------------

This role depends upon the the [`pulp_common`](../helper_roles/pulp_common) role, which in turn depends on on [pulp_repos](../helper_roles/pulp_repos). You should consult [`pulp_common`](../helper_roles/pulp_common) in particular for variables that effectively control the behavior of this role.

This role also utilizes some of the pulp_common role's variables in its logic:

* `pulp_django_admin_paths`
* `pulp_settings_file`
* `pulp_user`
* `pulp_user_home`
* `pulp_certs_dir`: Path where to generate or drop the keys for database fields encryption.
   Defaults to '{{ pulp_config_dir }}/certs' .
* `pulp_config_dir`
* `pulp_scripts_dir`: The collection/container signing service scripts exists under this directory
  with the filename `collection_sign.sh`/`container_sign.sh` when
  `galaxy_create_default_collection_signing_service==true`/
  `galaxy_create_default_container_signing_service==true`.

Like other roles that depend on pulp_common, if galaxy-ng is to be installed as a plugin, this role
will depending on the [`galaxy_post_install`](../helper_roles/galaxy_post_install). This role also
however utilizes some of the galaxy_post_install role's variables in its logic:

* `galaxy_create_default_collection_signing_service`
* `galaxy_create_default_container_signing_service`
* `pulp_settings.galaxy_collection_signing_service`
* `pulp_settings.galaxy_container_signing_service`

This role understands how to talk to the database server via `pulp_settings_file`,
which is written to disk in the `pulp_common` role, and whose relevant
values are set via the following variables:

* `pulp_settings.databases.default`: See pulp_database README.

Limitations
-----------

* pulp_database_config is designed to be run against only 1 host.

If it is accidentally run against multiple hosts (which will happen if
pulp_services is run against multiple hosts), 1 host will be picked to
actually run the tasks in pulp_database_config.

* pulp_database_config must be run against an existing host in a cluster if the
cluster is being expanded with this ansible playbook run.

For example, if you run pulp_database_config against host1 and host2, and you
later re-run pulp_installer to add host3 to the cluster, then either host1
or host2 must have pulp_database_config run against it during the re-run.
