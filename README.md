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

 * `users_root_password_salt` (string, default: *unset*):
   Salt to be used for hashing the root password.
   **Note**: Only required if `users_root_password` is set and
   `users_root_password_is_hashed` is false.

 * `users_customer_group` (string):
   Name of the system group to which all customer user accounts are added.
   **Note**: Only required if `users_customer` is non-empty.

### Optional

 * `users_root_password` (string, default: *unset*):
   If this is unset, the root password is not changed.
   If this is set and `users_root_password_is_hashed` is false, this is the
   password in clear-text, and `users_root_password_salt` must also be set.
   If this is set and `users_root_password_is_hashed` is true, this is assumed
   to be a hashed password (as produced by
   [`ansible.builtin.password_hash`][ansible:filter:password_hash]).

 * `users_root_password_is_hashed` (boolean, default: `false`):
   If set to `true`, `users_root_password` is assumed to have been hashed
   already (in this case, `users_root_password_salt` is not required).

 * `users_root_authorized_keys` (list, default: `[]`):
   SSH public keys that will be given authorisation to log in as `root`.
   Each list element is an object with the following properties:
    - `key` (string, mandatory):
      Key data itself.
    - `comment` (string, optional, default: *unset*):
      Comment, appended to the key line (usually `user@host`).
    - `description` (string, optional, default: *unset*):
      Human-readable description to be placed as a comment in the
      `authorized_keys` file above the key line.
    - `options` (string, optional, default: *unset*):
      Key options string to be prepended to the key line.

 * `users_adfinis` (list, default: `[]`):
   Adfinis user accounts to be set up. Each user will be added to the
   `{{users_adfinis_group}}` system group. Conversely, **every existing
   non-system user in that group that is not listed in this variable will be
   deleted**.
   Each list element is an object with the following properties:
    - `username` (string, mandatory):
      User account name.
    - `authorized_keys` (list, default: `[]`):
      SSH public keys that will be given authorisation to log in as `root`.
      Each list element is an object with properties as described in
      `users_root_authorized_keys`.

 * `users_adfinis_group` (string, default: `adfinis`):
   Name of the system group to which all Adfinis user accounts are added.

 * `users_adfinis_ssh_pubkey_options` (string, default: *unset*):
   Key options string to be prepended to *all* key lines.

 * `users_adfinis_homedir_mode` (file permission mode, default: `0700`):
   File permission mode for the home directory of each Adfinis user.
   **Note**: Because of a historical issue with Jinja2, the octal representation
   of the mode must either be passed as string (to ensure it is not incorrectly
   transformed), or [this Ansible option][ansible:vars:default_jinja2_native]
   must be set to true.

 * `users_adfinis_unrestricted_sudo` (boolean, default: `true`):
   Whether or not the Adfinis users are given unrestricted `sudo` permissions.

 * `users_adfinis_user_remove_home` (boolean, default: `false`):
   Whether or not to delete the home directory as well when deleting an unlisted
   Adfinis account.

 * `users_customer` (list, default: `[]`):
   Adfinis user accounts to be set up. Each user will be added to the
   `{{users_customer_group}}` system group.
   Each list element is an object with properties as described in
   `users_adfinis`.

 * `users_customer_homedir_mode` (file permission mode, default: `0755`):
   File permission mode for the home directory of each customer user.
   The default keeps it world-readable so that customers can use `sudo -u` to
   run commands as other users and still pass files in their home directory.
   **Note**: Because of a historical issue with Jinja2, the octal representation
   of the mode must either be passed as string (to ensure it is not incorrectly
   transformed), or [this Ansible option][ansible:vars:default_jinja2_native]
   must be set to true.

 * `users_customer_unrestricted_sudo` (boolean, default: `false`):
   Whether or not the customer users are given unrestricted `sudo` permissions.

 * `users_default_user` (string, default: `adfinis`):
   Name of initially existing non-root user account on system, to be deleted.

 * `users_default_user_remove_home` (boolean, default: `false`):
   Whether or not to delete the initially existing non-root user account's
   home directory as well.


[ansible:vars:default_jinja2_native]: https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-jinja2-native
[ansible:filter:password_hash]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/password_hash_filter.html
