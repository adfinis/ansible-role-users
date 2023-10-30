Contribution Guidelines
=======================

This guide attempts to establish a set of rules to follow when contributing to
this repository.


Branches
--------

### Development

Development happens around one primary branch (currently `master`).

However, changes must not be pushed to master directly (in fact, this is
prohibited). Instead, as a contributor, please create a new development branch
to work on, and open a *Pull Request* to have your changes reviewed and merged
into master.

We avoid merge commits as they may lead to unexpected/untestable issues and a
complicated history. Instead, the feature branch is fast-forward-merged into
master.

> #### Note
> There is an issue with [GitHub applying some auto-rebasing and rewriting magic
> that prevents this from working
> correctly](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github#rebasing-and-merging-your-commits).
> The consequence is diverging histories and unverified commits. This is a known
> issue.

For contributors, this means:

 * Rebase often.
 * Rewrite your history often to keep it clean.

### Releases

This repository follows [trunk-based
development](https://trunkbaseddevelopment.com/). This means that development
happens in one primary branch, and releases are then branched off that primary
branch as *release branches*. For further details on the concept, please see the
linked website.

Release versions follow [semantic versioning](https://semver.org/). In the
context of this repo, this means that backwards-incompatible/breaking changes
get a major version bump, backward-compatible non-bugfix changes get a minor
version bump, and backward-compatible bugfixes get a point release.

> #### Note
> We currently only do minor (pre)releases. Hence, all non-bugfix changes get a
> minor 0.x release, no matter if backward-compatible or not.

A release in this repository therefore happens in one of two ways:

 * **Regular release**:<br />
   A new set of features and/or changes is released in a new minor (or major)
   release:
    - [Update the changelog file](#changelog) to reflect the release version and
      date, and commit (or get it merged into master) as
      `feat(doc): Release vX.Y.0`.
    - Create a release branch at that commit with `vX.Y` (this covers all future
      point releases for that major/minor release) and push the branch.
    - Tag the commit with `vX.Y.0` and push the tag (this step cannot be
      reverted!)

 * **Bugfix release**:<br />
   A set of (backward-compatible) bugfixes is released in a point release:
    - `git checkout` the release branch to which you wish to backport the
      change.
    - `git cherry-pick -n` the commit(s) from the master branch (the `-n` option
      is likely needed for adapting the changelog entries before accepting the
      commit to the release branch), and push the changes.
    - Once all changes are applied, update the changelog file to reflect the
      release version and date, and commit as `feat(doc): Release vX.Y.Z`.

### Deprecation

We currently only support the latest release (see the [support
policy](README.md#support-policy)). This may change in the future.

This means that once a new minor/major version is released, the previous release
no longer receives any bugfixes or support.

In Git terms, this is achieved by **deleting the previous release branch**. Note
that individual release commits on those branches are not lost this way (they
are tagged and therefore still referenced).


Documentation
-------------

### `README.md`

The [README file](README.md) must contain the following sections:

 * A brief description of what the role is and does.

 * **Requirements**:<br />
   A description of assumptions made about the environment where the role is
   applied.<br />
   Note that this section does *not* describe roles/collections dependencies
   (see next section).

 * **Role Dependencies**:<br />
   If applicable, this contains two subsections (otherwise, simply "*(none)*"):
    - **Roles**: A list of roles that must be present. Roles that must be run by
      the playbook author beforehand must be marked as such.
    - **Collections**: A list of collections that must be present on the Ansible
      controller.

 * **Role Variables**:<br />
    - **Mandatory**: List of variables that *must* be set to use this role (to
      avoid undefined/failing behaviour). The format is as follows:
      ```markdown
       * `variable_name` (type):<br />
         Description goes here (adhere to 80-column line length).<br />
         You may use multiple lines if it helps readability.<br />
         **Note**: Put special remarks on separate lines below the main text.
      ```
      For list/dict variables, the format varies a bit:
      ```markdown
       * `variable_name` (list):<br />
         Description goes here again. Put a break before the next line.<br />
         Each element is an object with the following properties:

          - `keyname` (type):<br />
            Description of key (same format as main variables).
      ```
    - **Optional**: List of variables that need not be set, as there is a
      `default` variable defined. The format is the same as for mandatory
      variables (see above), but the default value must also be declared:
      ```markdown
       * `variable_name` (type, default: `…`):<br />
         …
      ```

 * **Role Tags**:<br />
   A list of Ansible tags that are respected by this role (and what effect they
   have).

Optionally, more sections can be added as needed.

### `CHANGELOG`

This file gives playbook authors an overview of the changes have happened
between releases.

The formatting is as follows:

```
(unreleased)

Changes that have not made it into any tagged release yet get this title, and
they are typically at the top.

Note that this section here is a human-readable description of the release
(before the enumeration of the changes start below), but this section is usually
not written before a release happens (or at all).

The enumeration of the changes then follows. The individual changes should be
written as imperative (similar to Git commits, though without Conventional
Commits prefixes, and may wrap over multiple lines if needed).

Note that there is no need to put a section for which there is no entry (you
will rarely have all of these categories at once). The order must be kept this
way!

Changes:

 - Remove featX deprecated in vX.Y.Z.
 - Have option foobar delete system instead of updating it (breaking
   change!).
 - …

Bugfixes:

 - Have option foobaz check system instead of deleting it.
 - …

Improvements:

 - …

Additions:

 - …

Documentation:

 - …

Miscellaneous:

 - …

--------------------------------------------------------------------------------

v42.3.1 (released: 2099-10-30)

Releases get a section like this. Note the simple dashed line above to separate
individual POINT releases.

… (enumeration of changes)

--------------------------------------------------------------------------------

v42.3.0 (released: 2099-10-29)

… (enumeration of changes)

================================================================================

v42.2.0 (released: 2099-09-01)

Regular minor and major releases are separated by a double dashed line (i.e.
with "="s).

… (enumeration of changes)

================================================================================

… (older releases)
```

Note that a changelog will always only contain the changes that have led up to
the release, and none of the changes in other branches.

In the master branch, this means that there is never a point release changelog
entry (and thus never a single dashed line); and in a release branch, there is
only ever point release entries for that one release branch, but none of the
previous releases.

To get a better understanding of the release process, see the section
[Releases](#Releases).
