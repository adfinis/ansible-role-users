users
=====

This role sets up customer and Adfinis user accounts.


Requirements
------------

This role assumes that there is an initial non-root user with sudo permissions
present on the system (`{{users_default_user}}`, see *Role Variables* below).

That user is used for the initial user accounts setup and then deleted. The
deletion happens in the last task of this role (so an initial run will work
fine, but if repeated, login will fail).

In a playbook (or series of playbook), it is therefore recommended to apply this
role in two variants:

 1. The first time, apply with `remote_user: {{users_default_user}}`.

 2. After than, apply with the intended user (either the personal account, or
    `root`, if allowed).

It is recommended to keep a playbook/play around for the initial setup, and a
playbook/play for continuous management.


Role Dependencies
-----------------

*(none)*


Role Variables
--------------

### Mandatory

 * `users_root_password_salt` (string, default: *unset*):<br />
   Salt to be used for hashing the root password.<br />
   **Note**: Only required if `users_root_password` is set and
   `users_root_password_is_hashed` is false.

 * `users_customer_group` (string):<br />
   Name of the system group to which all customer user accounts are added.<br />
   **Note**: Only required if `users_customer` is non-empty.

### Optional

 * `users_root_password` (string, default: *unset*):<br />
   If this is unset, the root password is not changed.<br />
   If this is set and `users_root_password_is_hashed` is false, this is the
   password in clear-text, and `users_root_password_salt` must also be set.<br
   />
   If this is set and `users_root_password_is_hashed` is true, this is assumed
   to be a hashed password (as produced by
   [`ansible.builtin.password_hash`][ansible:filter:password_hash]).

 * `users_root_password_is_hashed` (boolean, default: `false`):<br />
   If set to `true`, `users_root_password` is assumed to have been hashed
   already (in this case, `users_root_password_salt` is not required).

 * `users_root_authorized_keys` (list, default: `[]`):<br />
   SSH public keys that will be given authorisation to log in as `root`.<br />
   Each list element is an object with the following properties:
    - `key` (string, mandatory):<br />
      Key data itself.
    - `comment` (string, optional, default: *unset*):<br />
      Comment, appended to the key line (usually `user@host`).
    - `description` (string, optional, default: *unset*):<br />
      Human-readable description to be placed as a comment in the
      `authorized_keys` file above the key line.
    - `options` (string, optional, default: *unset*):<br />
      Key options string to be prepended to the key line.

 * `users_adfinis` (list, default: `[]`):<br />
   Adfinis user accounts to be set up. Each user will be added to the
   `{{users_adfinis_group}}` system group. Conversely, **every existing
   non-system user in that group that is not listed in this variable will be
   deleted**.<br />
   Each list element is an object with the following properties:
    - `username` (string, mandatory):<br />
      User account name.
    - `authorized_keys` (list, default: `[]`):<br />
      SSH public keys that will be given authorisation to log in as `root`.<br
      />
      Each list element is an object with properties as described in
      `users_root_authorized_keys`.

 * `users_adfinis_group` (string, default: `adfinis`):<br />
   Name of the system group to which all Adfinis user accounts are added.

 * `users_adfinis_ssh_pubkey_options` (string, default: *unset*):<br />
   Key options string to be prepended to *all* key lines.

 * `users_adfinis_homedir_mode` (file permission mode, default: `0700`):<br />
   File permission mode for the home directory of each Adfinis user.<br />
   **Note**: Because of a historical issue with Jinja2, the octal representation
   of the mode must either be passed as string (to ensure it is not incorrectly
   transformed), or [this Ansible option][ansible:vars:default_jinja2_native]
   must be set to true.

 * `users_adfinis_unrestricted_sudo` (boolean, default: `true`):<br />
   Whether or not the Adfinis users are given unrestricted `sudo` permissions.

 * `users_adfinis_user_remove_home` (boolean, default: `false`):<br />
   Whether or not to delete the home directory as well when deleting an unlisted
   Adfinis account.

 * `users_customer` (list, default: `[]`):<br />
   Adfinis user accounts to be set up. Each user will be added to the
   `{{users_customer_group}}` system group.<br />
   Each list element is an object with properties as described in
   `users_adfinis`.

 * `users_customer_homedir_mode` (file permission mode, default: `0755`):<br />
   File permission mode for the home directory of each customer user.<br />
   The default keeps it world-readable so that customers can use `sudo -u` to
   run commands as other users and still pass files in their home directory.<br
   />
   **Note**: Because of a historical issue with Jinja2, the octal representation
   of the mode must either be passed as string (to ensure it is not incorrectly
   transformed), or [this Ansible option][ansible:vars:default_jinja2_native]
   must be set to true.

 * `users_customer_unrestricted_sudo` (boolean, default: `false`):<br />
   Whether or not the customer users are given unrestricted `sudo` permissions.

 * `users_default_user` (string, default: `adfinis`):<br />
   Name of initially existing non-root user account on system, to be deleted.

 * `users_default_user_remove_home` (boolean, default: `false`):<br />
   Whether or not to delete the initially existing non-root user account's
   home directory as well.


Role Tags
---------

 * `init`: Same as `role::users:root` and `role::users:adfinis` combined.
 * `role::users`: All tasks in this role.
 * `role::users:root`: All tasks that set up the root user account.
 * `role::users:adfinis`: All tasks that set up the Adfinis user accounts.
 * `role::users:adfinis:create`: All tasks that set up the Adfinis user accounts
   (without deleting).
 * `role::users:adfinis:delete`: All tasks that delete superfluous Adfinis
   user accounts.
 * `role::users:customer`: All tasks that set up customer user accounts.
 * `role::users:default`: All tasks that clean up the default user account.


Support Policy
--------------

Only the latest release is maintained and supported (see the [Tags
page](https://github.com/adfinis/ansible-role-users/tags)).

Once a new release is made, the previous release branch no longer receives any
bugfixes.


[ansible:vars:default_jinja2_native]: https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-jinja2-native
[ansible:filter:password_hash]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/password_hash_filter.html
