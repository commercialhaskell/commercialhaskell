## Improved Hackage security

This page is intended to collect information about:

* Current shortcomings of the Hackage/cabal-install combination from a security perspective
* Possible short term mitigations of these security flaws
* Long term strategies for more secure systems

### Prior work

This page is built on both prior proposals and discussions in mailing lists. If
there is a discussion or proposal that is not mentioned, please add it here
(and try to keep it chronoligcal).

* [Package signing detailed proposal](https://github.com/commercialhaskell/commercialhaskell/wiki/Package-signing-detailed-propsal) Wiki page
* [Our composable community infrastructure](https://www.fpcomplete.com/blog/2015/03/composable-community-infrastructure) blog post
    * [Reddit discussion](http://www.reddit.com/r/haskell/comments/30dfxi/fpcomplete_blog_our_composable_community/)
* [Improvements to package hosting and security](https://groups.google.com/d/msg/commercialhaskell/PTbC0p_YFvk/8XqS8wDxgqEJ) mailing list thread
* [Improving Hackage Security](http://www.well-typed.com/blog/2015/04/improving-hackage-security/) blog post
    * [Reddit discussion](http://www.reddit.com/r/haskell/comments/32sezy/ongoing_work_to_improve_hackage_security/)

### Current shortcomings

* Reliability: if Hackage goes down, no one can download packages or the package index
    * From a security standpoint: we can't currently have untrusted mirrors due to lack of any hash/signature mechanism
* Single point of failure: if Hackage (either the software of the hosting infrastructure) is hacked, the attacker has full ability to distribte malicious code
* Man in the middle (download side): since traffic between cabal-install and Hackage is plain text, any man in the middle may inject malicious code to the downloaded packages
* Man in the middle (upload side): when uploading a package, traffic is also unencrypted. A man in the middle may replace the payload with nefarious code and compromise the entire package database
* Eavesdropping: uploads are secured by HTTP digest security over HTTP. When performing uploads from an unsecured location (e.g., open WiFi in an airport), those headers can be sniffed and possibly spoofed
* Password compromise: standard weaknesses of any password based system regarding password stealing/guessing/social engineering
* There is little to no insight into how a package (or revision of a cabal file) become accepted into Hackage, and therefore external verification of the index or tarballs is all but impossible
    * We have a Hackage username for each upload/revision, but there's no way to verify ownership of a Hackage username besides trying to log into the system
    * We don't know *why* an action was alloed. Is the user an admin? a maintainer? a trustee? How can we see the historical log of changes to these statuses?

### Short-term fixes

These fixes are not intended to completely address any problem, simply to mitigate existing attack vectors

* Hackage should add SHA512 hashes for each tarball to the 00-index file so that downloads can be verified
* cabal-install should be modified to use HTTPS instead of HTTP

### Long-term solutions

There are various ideas at play already. The bullets are not intended to be
full representations of the proposals, but rather high level summaries. We
should continue to expand this page with more details going forward.

I am *not* including one proposal I'm aware of since it hasn't been made
publicly yet. The relevant parties should feel free to add that proposal (and
others I'm unaware of).

Also note: despite the tone of some of the above-linked discussions, proposals
here are not necessary mutually exclusive.

* Add cryptographic signing of packages. Authors will sign their packages and store the signatures in a publicly downloadable place. End users can then verify those signatures. (Tooling would automated this.)
* Separate out various pieces of infrastructure, such as package hosting, index hosting, uploading, a web interface, etc, to simplify, allow us to make the core required components more robust, and reduce attack surface area on those components
* Make all actions around packages a publicly viewable log via a Git repository. All actions are stored in files which are signed by relevant parties. A central authority would delegate permissions to individuals to do different things, such that the central authority has relatively few actions to take, and individual key compromises do not affect the entire ecosystem, and can be mitigated by revoking rights.
* Add support for "The Update Framework" to Hackage (apologies, I'm hazy on the details of what is actually proposed, hopefully others in the know will fill this in).
