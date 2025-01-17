Code management: scc
====================

scc is a Python library with a set of utility commands used for code
management and used in the OME :doc:`continuous-integration`.
More information can be found in the :pypi:`Python package page <scc>` or  in
the `source code page <https://github.com/ome/snoopycrimecop>`_.

If you find a bug or if you want an additional feature to be implemented,
please `open an issue <https://github.com/ome/snoopycrimecop/issues>`_.

Installation
------------

The scc tools are a set of Python_ based utility programs. The tools suite
can be installed using :command:`pip`::

    $ pip install -U scc

This command will install and/or upgrade the **PyGithub** and **yaclifw**
package dependencies. If the version of Python installed is older than 2.7,
this may also install the **argparse** package.

Github connection
-----------------

.. _About Two-Factor Authentication: https://help.github.com/articles/about-two-factor-authentication/
.. _Creating an access token for command-line use: https://help.github.com/articles/creating-an-access-token-for-command-line-use/

Most of the scc commands instantiate a Github connection using the PyGithub
package. GitHub strongly recommends to turn on two-factor authentification
(2FA), see `About Two-Factor Authentication`_ for more details. If 2FA is
activated, the only way to use scc commands creating a GitHub connection is to
create an OAuth token, see
`Creating an access token for command-line use`_
for details on how to create Personal Access Tokens via the GitHub interface.
This token can then be stored in the global Git configuration file::

    git config --global github.token REPLACE_BY_PERSONAL_ACCESS_TOKEN

Unless the ``--token`` option is passed to the scc command, the
command first looks for the ``github.token`` specified in the git config file
and, if found, uses this token to connect to GitHub::

    $ scc merge master --info -v
    2013-01-16 22:03:49,633 [  scc.config] DEBUG Found github.token
    ...

If no token is found, the command looks for a ``github.user`` in the git config
file and, if found, uses this username to connect to Github::

    $ scc merge master --info -v
    2013-01-16 22:06:00,256 [  scc.config] DEBUG Found github.user
    Enter password for https://github.com/sbesson:

.. note::
   The password to be entered here is the GitHub password. Connecting using
   the GitHub username/password is NOT possible if 2FA has been activated.

Finally, if no token or user is found, both the GitHub username and password
are queried at the prompt::

    $ scc merge master --info -v
    # github.token and github.user not found.
    # See `scc token` for simpifying use.
    Username or token: sbesson
    Enter password for https://github.com/sbesson:

.. _scc merge:

scc merge
---------

Merge all the PRs based on specified branch matching the input filters
including all submodules.

Description
^^^^^^^^^^^

Filters of different types can be specified and combined to include and
exclude a set of PRs in the merge command. Each filter needs to be formatted
as ``key:value``. If no key but only a value is specified, it is assumed the
filter is a label filter (see below). These filters can be passed to an
:option:`scc merge --include` or :option:`scc merge --exclude` option.

The available filter types are described below:

- Label filters can be specified using the ``label`` key i.e.
  ``label:<LABEL>``. This filter type will match a Pull Request if one of the
  following conditions is met:

  1. a label named ``<LABEL>`` is applied to the Pull Request
  2. the Pull Request description contains a line starting with ``--<LABEL>``
  3. one of the Pull Request comments contains a line starting with
     ``--<LABEL>``. Note this comment needs to be written by one of the public
     members of the organization owning the upstream repository.

- User filters can be specified using the ``user`` key i.e. ``user:<USER>``.
  This filter type will select a Pull Request if it has been opened by the
  user ``USER``. Additionally, two special user values are allowed:

  1. the ``#org`` value will match all PRs opened by public members of the
     organization of the upstream repository
  2. the ``#all`` value will match all PRs opened by any user

- PR filters can be specified using the ``pr`` key i.e. ``pr:<NUMBER>``. This
  will select Pull Requests whose ID matches the input number. The form
  ``#number`` is also recognized as a PR filter. For repositories containing
  submodules, it is possible to filter submodule PRs using
  ``user/repo#number``.

Arguments
^^^^^^^^^

The first argument is the name of the base branch of origin, e.g.::

    $ scc merge develop

.. program:: scc merge

.. option:: --comment

    Add a comment to the PR if there is a conflict while merging the PR
    ::

        $ scc merge develop --comment

.. option:: --default <filterset>, -D <filterset>

    Specify the default set of filters to use

    Three filter sets are currently implemented: ``none``, ``org`` and
    ``all``. The ``none`` filter set has no preset filter. The ``org`` filter
    set uses ``user:#org`` and ``label:include`` as the default include
    filters and ``label:exclude`` and ``label:breaking`` as the default
    exclude filters. The ``all`` filter set uses ``user:#all`` as the default
    include filters.

    Default: ``org``

.. option:: --exclude <filter>, -E <filter>

    Exclude PR by filter (see filter semantics above)::

        $ scc merge develop -E label:l1 -E user:u1 -E #45 -E org/repo#40

.. option:: --include <filter>, -I <filter>

    Include PR by filter (see filter semantics above)::

        $ scc merge develop -I label:l1 -I user:u1 -I #45 -I org/repo#40

.. option:: --check-commit-status <status>, -S <status>

    Exclude PR based on the status of the last commit

    Three options are currently implemented: ``none``, ``no-error`` and
    ``success-only``. By default (``none``), the status of the last commit on
    the PR is not taken into account.
    To include PRs which have a successful status only, e.g. PRs where the
    Travis build is green, use the ``success-only`` option::

        $ scc merge develop -S success-only

    To exclude all PRs with an ``error`` or ``failure`` status, use the
    ``no-error`` option::

    $ scc merge develop -S no-error

.. option:: --info

    Display the candidate PRs to merge but do not merge them
    ::

    $ scc merge develop --info

.. option:: --push <branchname>

    Push the locally merged branch to Github
    ::

        $ scc merge develop --push my-merged-branch

.. option:: --reset

    Recursively reset each repository to the HEAD of the base branch
    ::

        $ scc merge develop --reset

.. option:: --shallow

    Merge the PRs for the top-level directory only, excluding submodules::

        $ scc merge develop --shallow

.. option:: --remote <remote>

    Specify the name of the remote to use as the origin. Default: origin::

        $ scc merge develop --remote gh

As a concrete example, the first step of a merge job is calling the following merge command::

    $ scc merge master --no-ask --reset --comment --push merge_ci

Use cases
^^^^^^^^^

The basic command will use the default filters and merge all PRs opened
against ``master`` by any public members of the organization, include any PR
labeled as ``include`` and exclude any PR labeled as ``breaking`` or
``exclude``::

   $ scc merge master

The following command overrides the default set of filters and will only merge
PRs opened against ``master`` labeled as ``my_label``::

    $ scc merge master -Dnone -Ilabel:my_label

The following command overrides the default set of filters and will merge all
PRs opened against ``master`` by public members of the organization, include
any PR labeled with ``my_label`` and exclude any PR labeled as ``exclude``::

    $ scc merge master -Dnone -Iuser#org -Ilabel:my_label -Elabel:exclude

.. versionchanged:: 0.3.0
    Added default values for :option:`--include` and :option:`--exclude`
    options.

.. versionchanged:: 0.3.8
    Added :option:`--shallow` and :option:`--remote` options.

.. versionchanged:: 0.4.0
    Added :option:`--check-commit-status` option.

scc travis-merge
----------------
.. program:: scc travis-merge

Merge PRs in a Travis environment, using the PR comments to generate the merge
filters.

::

    $ scc travis-merge

This command internally defines all the filter options exposed in
:program:`scc merge`.

The target branch is read from the base of the PR, the
:option:`scc merge --default` option is set to ``none`` meaning no PR is
merged by default and no default :option:`scc merge --exclude` option is
defined.

The :option:`scc merge --include` filter is determined by parsing all the PR
comments lines starting with ``--depends-on``.

To include a PR from the same GitHub repository, use the PR number prepended
by ``#``. For instance, to include PR 67 in the Travis build, add a comment
line starting with ``--depends-on #67`` to the PR.

To include a PR from a submodule, use the PR number prepended by
``submodule_user/submodule_name#``. For instance, to include PR 60 of
bioformats in the Travis build, add a comment line starting with
``--depends-on openmicroscopy/bioformats#60`` to the openmicroscopy PR.

.. note::
  The :program:`scc travis-merge` command works solely for Pull Requests'
  Travis builds.

scc update-submodules
---------------------

.. program:: scc update-submodules

Update the pointer of all submodules based on specified branch.

The first argument is the name of the base branch of origin, e.g.::

    $ scc update-submodules develop

.. option:: --push <branchname>

    Push the locally merged branch to Github and open a PR against the base
    branch::

        $ scc merge develop --push submodules_branch

.. option:: --no-pr

    Combined with :option:`--push` option, push the locally merged branch to
    Github but skip PR opening::

        $ scc merge develop --push submodules_branch --no-pr

.. option:: --remote <remote>

    Specify the name of the remote to use as the origin (default: origin)::

        $ scc update-submodules develop --remote gh


scc rebase
----------
.. program:: scc rebase

Rebase a PR (open or closed) onto another branch and open a new PR.

The first argument is the number of the PR to rebase and the second argument
is the name of the branch onto which the PR should be rebased::

    $ scc rebase 142 develop

Assuming the head branch used to open the PR 142 was called ``branch_142``,
this command will rebase the tip of ``branch_142`` onto origin/develop, create
a new local branch called ``rebased/develop/branch_142``, push this branch to
Github and open a new PR. Assuming the command opens PR 150, to facilitate the
integration with :program:`scc check-prs`, a ``--rebased-to #150``
comment is added to PR 142 and a ``--rebased-from #142`` comment is
added to the PR 150. Finally, the command will switch back to the original
branch prior to rebasing and delete the local ``rebased/develop/branch_142``.

.. note::

    By default, :program:`scc rebase` uses the branches of the ``origin``
    remote to rebase the PR. To specify another remote, use the
    :option:`--remote` option.

.. option:: --no-pr

    Skip the opening of the PR
    ::

        $ scc rebase 142 develop --no-pr

.. option:: --no-delete

    Do not delete the local rebased branch

    ::

        $ scc rebase 142 develop --no-delete

.. option:: --remote <remote>

    Specify the name of the remote to use for the rebase (default: origin)

    ::

        $ scc rebase 142 develop --remote snoopycrimecop

.. option:: --continue

    Re-run the command after manually fixing conflicts

    If :program:`scc rebase` fails due to conflict during the rebase, you will
    end up in a detached HEAD state.

    If you want to continue the rebase operation, you will need to manually
    fix the conflicts::

        # fix files locally
        $ git add conflicting_files # add conflicting files
        $ git rebase --continue

    This conflict solving operation may need to be repeated multiple times
    until the branch is fully rebased.

    Once all the conflicts are resolved, call :program:`scc rebase` with the
    :option:`--continue` option::

        $ scc rebase --continue 142 develop

    Depending on the input options, this command will perform all the steps of
    the rebase command (Github pushing, PR opening) skipping the rebase part.

    Alternatively, you can abort the rebase and switch to your previous branch::

        $ git rebase --abort
        $ git checkout old_branch

.. versionchanged:: 0.3.10
    Automatically add ``--rebased-to`` and ``--rebased-from`` comments to the
    source and target PRs.

scc check-prs
-------------
.. program:: scc check-prs

Compare two development branches and check that PRs merged in one branch have
been merged to the other.

The basic workflow of the :program:`scc check-prs` command is the
following:

-   list all first-parent merge commits for each branch including git notes
    referenced as ``see_also/other_branch`` where other_branch is the name of
    the branch to check against.
-   exclude all merge commits with a note containing either "See gh-" or "n/a"
-   for each remaining merge commit, parse the PR number and look into the PR
    body/comments for lines starting with ``--rebased-to``, ``--rebased-from``
    or ``--no-rebase``.

Additionally, for each line of each PR starting with ``--rebased-to`` or
``--rebased-from``, the existence of a matching line is checked in the
corresponding source/target PR. For instance, if PR 70 has a
``--rebased-from #67`` line and a ``--rebased-from #66`` line, then both PRs
66 and 67 should have a ``--rebased-to #70`` line.

This command requires two positional arguments corresponding to the name of
the branch of origin to compare::

    $ scc check-prs dev_4_4 develop

.. option:: --shallow

    Check PRs in the top-level directory only, excluding submodules::

        $ scc check-prs dev_4_4 develop --shallow

.. option:: --remote <remote>

    Specify the name of the remote to use as the origin (default: origin)::

        $ scc check-prs dev_4_4 develop --remote gh

.. option:: --no-check

    Do not check links between rebased comments::

        $ scc check-prs dev_4_4 develop --no-check

.. versionadded:: 0.3.10
    Added support for body/comment parsing and ``--rebased-to/from``
    linkcheck

.. versionchanged:: 0.4.0
    Improved command output and added support for submodule processing

.. versionchanged:: 0.5.0
    Renamed command

scc version
-----------
.. program:: scc version

Return the version of the scc tools::

    $ scc version
    0.3.0

.. versionadded:: 0.2.0

.. _scc deploy:

scc deploy
----------

.. program:: scc deploy

Deploy a website update using `file symlink replacement <https://gist.github.com/datagrok/3807742#file-symlink-replacement-md>`_::

    $ scc deploy folder

The goal of this command is to enable overwriting of deployed doc content and
allow for “hot-swapping” content served by Apache without downtime and HTTP
404s.

.. option:: --init

    Prepare folder for symlink replacement. Should only be run once

    ::

        $ scc deploy folder --init

.. versionadded:: 0.3.1

The hudson jobs ending with ``release-docs`` and OMERO-docs-internal deploy the
documentation artifacts to necromancer. The target directory (sphinx-docs) is
controlled by the hudson:hudson user, so all file system operations are
allowed. Each job has the target directory configured in the SSH publisher
target directory property. After deployment has happened to a temporary
directory, a series of symlink moves happens making sure that the symlink
points to the updated content.

scc check-status
----------------
.. program:: scc check-status

Check the status of the Github API::

    $ scc check-status && echo "Passing"
    Passing

.. option:: -n <N>

    Display N last status messages from Github API history::

        $ scc check-status -n 4
        2013-11-04 13:40:48 (minor) We're investigating an increase in error responses from the API.
        2013-11-04 14:33:55 (good) Everything operating normally.
        2013-11-05 12:59:50 (minor) We're investigating reports of an increase in 502s from the GitHub API.
        2013-11-05 13:07:15 (good) Everything operating normally.


.. versionadded:: 0.4.0
