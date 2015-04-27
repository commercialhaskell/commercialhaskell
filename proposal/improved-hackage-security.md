## Improved Hackage security

This page is intended to collect information and coordinate actions around the following topics:

* Security concerns regarding the current Hackage/cabal-install service/toolchain combination
* Possible short term mitigations of these security flaws
* Long term strategies for more secure systems

### Prior work

Hackage and security has been a recurring topic of conversation on mailing lists.
Proposals have already been formulated to address specific threats.
The list below is an inventory of such discussions.
If there is a discussion or proposal that is not mentioned, please add it here (and try to keep it chronoligcal).

* [Package signing detailed proposal](https://github.com/commercialhaskell/commercialhaskell/wiki/Package-signing-detailed-propsal) Wiki page (authors: Chris Done)
* [Our composable community infrastructure](https://www.fpcomplete.com/blog/2015/03/composable-community-infrastructure) blog post (authors: Mathieu Boespflug)
    * [Reddit discussion](http://www.reddit.com/r/haskell/comments/30dfxi/fpcomplete_blog_our_composable_community/)
* [Improvements to package hosting and security](https://groups.google.com/d/msg/commercialhaskell/PTbC0p_YFvk/8XqS8wDxgqEJ) mailing list thread (authors: many)
* [Improving Hackage Security](http://www.well-typed.com/blog/2015/04/improving-hackage-security/) blog post (authors: Duncan Coutts and Austin Seipp)
    * [Reddit discussion](http://www.reddit.com/r/haskell/comments/32sezy/ongoing_work_to_improve_hackage_security/)

### High level goals

There are approximately three goals that various people are trying to achieve.
These goals are interconnected, but decompose fairly nicely:

1. Higher availability and more reliable hosting of our community infrastructure (in particular package tarballs and the package index).
2. Certification of provenance, to guarantee that a package tarball really was created by a trusted party (this is what package signing by authors is about).
3. Certification of freshness (this is what package index signing is about), to
  1. guarantee that package updates really are the latest ones including fixes to all known vulnerabilities, or to detect when this is not the case;
  2. make it possible to guarantee that package data and metadata served by untrusted mirrors is the same as that served by Hackage itself (modulo a small lag window).

There's been quite a bit of confusion around (2) and (3). While solutions to
these can overlap, the goals are quite different, and conflation of these two
goals seems to have led to a lot of miscommunications in this discussion. To try and make the distinction quite clear:

* When authors sign packages, it is possible to verify the a single package was in fact signed off on by a specific person. This prevents an intermediate party from injecting a different package in its place. For example, if Edward Kmett signs lens-4.9, a user can verify that the lens he/she downloaded from Hackage was in fact signed by Edward, and was not a forgery uploaded by someone else.
* With index signing, a user can tell whether `cabal update` will pull the latest set of package metadata, whether from Hackage or a mirror. A user can also tell whether an attacker has attempted to roll back Hackage to a previous version. This is important because a MITM could in theory block a user from seeing the latest package metadata, which might include updates to known vulnerabilities that the attacker wants to avoid the user from installing. Further, index signing makes it possible to trust downloads from any mirror *as much as (but no more than)* downloads from Hackage, even when downloading package tarballs that were not signed by the author. However, it does not fully protect against a Hackage compromise. If Hackage is compromised, then an attacker could still inject malicious package tarballs if the original authors didn't sign all their uploads.

To be clear, these two signing systems *complement* each other instead of competing with each other. For example, with only package signing, a nefarious server could simply not provide a bugfixed version of lens, and continue to distribute an older (signed) version with a known vulnerability. With only index signing, there is no protection against Hackage being compromised, or against a MITM attack on the package author while he/she is uploading a new package tarball to Hackage.

### Current shortcomings

* Reliability: if Hackage goes down, no one can download packages or the package index
    * From a security standpoint: we can't currently have untrusted mirrors due to lack of any hash/signature mechanism
* Single point of failure: if Hackage (either the software of the hosting infrastructure) is hacked, the attacker has full ability to distribute malicious code
* Man in the middle (download side): since traffic between cabal-install and Hackage is plain text, any man in the middle may inject malicious code to the downloaded packages
* Man in the middle (upload side): when uploading a package, traffic is also unencrypted. A man in the middle may replace the payload with nefarious code and compromise the entire package database
* There is no way of knowing whether the data being pulled from Hackage is really the latest. An attacker (via MITM or server compromise) could be serving legitimate unadelterated but old packages with known vulnerabilities, or preventing a user from seeing that there are later versions that fix some vulnerability.
* Either by compromising the server or via MITM, an attacker can convince clients to download arbitrary amounts of data, filling up their disk, or to hang potentially indefinitely (though the latter is a tooling problem).
* Eavesdropping: uploads are secured by HTTP digest security over HTTP. When performing uploads from an unsecured location (e.g., open WiFi in an airport), those headers can be sniffed and possibly spoofed
* Password compromise: standard weaknesses of any password based system regarding password stealing/guessing/social engineering
* No audit: there is little to no insight into how a package (or revision of a cabal file) becomes accepted into Hackage, and therefore external verification of the index or tarballs is all but impossible
    * We have a Hackage username for each upload/revision, but there's no way to verify ownership of a Hackage username besides trying to log into the system
    * Even then, we have no guarantee that this log hasn't been tampered with, either due to a bug or by an attacker.
    * We don't know *why* an action was allowed. Is the user an admin? a maintainer? a trustee? How can we see the historical log of changes to these statuses?

### Side goals

In addition to the above points, there are at least a
few other goals in the community right now, such as:

* Offline access to the package database, for situations where downloading from Hackage/an S3 mirror are impossible
* Efficient incremental synchronization of the package database (aka `cabal update`)
* Have some kind of versioning of the package database, to say things like "this is built on top of Hackage version X"

There are certainly other goals that people have in mind, and it would be great
to mention those explicitly so we can try to account for them when making any
decisions.

### Short-term fixes

These fixes are not intended to completely address any problem, simply to mitigate existing attack vectors

* Hackage should add SHA512 hashes for each tarball to the 00-index file so that downloads can be verified
* cabal-install should be modified to use HTTPS instead of HTTP
* Add a means via Hackage to verify identity of a user, either via a mechanism like OAuth, or by allowing users to add a GPG public key to their accounts that will be publicly viewable

### Long-term solutions

There are various ideas at play already. The bullets are not intended to be
full representations of the proposals, but rather high level summaries. We
should continue to expand this page with more details going forward.

Note: despite the tone of some of the above-linked discussions, proposals
here are not necessary mutually exclusive.

TODO: there is at least one proposal out there, that has not been listed here,
because the authors have not elected to make that proposal public yet.
The relevant parties should feel free to add that proposal (and
others I'm unaware of).

* Add cryptographic signing of packages. Authors will sign their packages and store the signatures in a publicly downloadable place. End users can then verify those signatures. (Tooling would automated this.)
* Separate out various pieces of infrastructure, such as package hosting, index hosting, uploading, a web interface, etc, to simplify, allow us to make the core required components more robust, and reduce attack surface area on those components
* Make all actions around packages a publicly viewable log via a Git repository. All actions are stored in files which are signed by relevant parties. A central authority would delegate permissions to individuals to do different things, such that the central authority has relatively few actions to take, and individual key compromises do not affect the entire ecosystem, and can be mitigated by revoking rights.
* Add support for "The Update Framework" (TUF) to Hackage.
    * Note that the initial work around TUF proposed by Well Typed only covers index signing, though there is a possibility to add package signing as well in the future.
