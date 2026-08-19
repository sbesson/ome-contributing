Security process
================

This document describes the security process for OME maintainers. The process for
reporting a security vulnerability is described in the
`security page <https://www.openmicroscopy.org/security/>`_ of the OME website.
This process makes extensive use of GitHub functionalities for managing vulnerability
reports and fixes - see
https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities.

Create draft advisory
^^^^^^^^^^^^^^^^^^^^^

Once a vulnerability is [reported](https://www.openmicroscopy.org/security/) through
the security mailing list, an administrator should draft a security advisory
against the appropriate GitHub repository e.g.
https://github.com/ome/omero-web/security/advisories. This advisory can be used for
private discussions about the report and the fix. Comments on an advisory will not
be available publicly but will remain available to the project maintainers.

The vulnerability report should first be reviewed by the project maintainers, acknowledging
its receipt to the report and engaging for additional information as necessary.

If the reported problem is not identified as a security risk and a draft advisory has
been created, it must be closed with a comment explaining why it is not considered.

Fix a reported vulnerability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the problem has been identified as a security risk, a temporary private fork of
the relevant repository should be created from the draft advisory, see
https://docs.github.com/en/code-security/tutorials/fix-reported-vulnerabilities/collaborate-in-a-fork/.

Collaborators can be added and removed from the private fork as described in 
https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/fix-reported-vulnerabilities/add-collaborators
and https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/fix-reported-vulnerabilities/remove-collaborators
as necessary.

The development and review process can then proceed as usual using pull requests
against the private fork.

Announce the security release
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once a fix has been identified, a pre-announcement of the upcoming security release
should be made on the image.sc forum including at least the `ome` and `security` tags.
The delay between the publication pre-announcement and the security release may vary
depending on a number of factors. For OMERO security releases, sufficient notice should
be given to system administrators to plan and schedule a downtime and upgrade.

Prepare the advisory
^^^^^^^^^^^^^^^^^^^^

Prior to its publication, the content advisory should be updated with:

- the package, affected versions and patched versions
- a description including Background, Impact, Workaround and Resolution
- an assessment of the severity using CVSS v3 base metrics
- a CVE, if an identifier has already been assigned as part of the vulnerability report,
  it should be re-used, otherwise, a CVE should be requested from the GitHub advisory
- credits to the reporter if applicable

Publish the advisory
^^^^^^^^^^^^^^^^^^^^

On release day, all open Pull Requests against the temporary fork should be merged
in the security advisory - see https://docs.github.com/en/code-security/tutorials/fix-reported-vulnerabilities/collaborate-in-a-fork#merging-changes-in-a-security-advisory
Once merged, all changes will be visible on the public repository.

The release process can then follow the standard procedures with the following variations:

- the GitHub advisory should be published as described in
  https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/fix-reported-vulnerabilities/publish-repository-advisory.
- in addition to the release announcement, the GitHub advisory should be duplicated to the
  `list of advisories <https://www.openmicroscopy.org/security/advisories/>`_ on the OME website.

