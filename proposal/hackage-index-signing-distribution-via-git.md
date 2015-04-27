# A signed and versioned object database for secure package distribution

Aka Just Use Git.

## Goals

Package distribution suffers greatly from lack of high availability:
every time Hackage goes offline, hacking on new projects grinds and
end users become frustrated. There are two ways to increase
availibility: **improve service reliability and increase redundancy**.
This is a concrete proposal for doing both, while simultaneously
improving the security of package distribution for end users and
developers.

Increasing redundancy means replicating the content served by Hackage
to mirrors around the world. This mitigates the impact of global
network outages, decreases latencies and lightens the load on Hackage
itself. However, how can one be sure that downloading package
information from a mirror yields the same content as downloading
directly from Hackage? How do we know that we're not downloading from
a phony mirror setup by an attacker to serve us vulnerable packages?
How do we know that no eavesdropper is silently serving us bogus data,
even as we're trying to contact a legitimate mirror, or Hackage
itself?

Consider Alice, a prolific Haskell developer who maintains a number of
packages on Hackage including `foolib`, Bob the end user and Mallory,
an evil attacker. End-to-end package signing makes it possible for Bob
to cautiously only build and install some version of `foolib` if he
knows for sure that it comes from Alice, provided Bob somehow knows
for sure which key Alice uses to sign her packages on Hackage.
However, end-to-end package signing serves only to check the
*provenance* of a package. Knowing the provenance of the packages that
one installs is not enough to avoid package based attacks on Bob's
machine, no matter whether Bob gets his packages from Hackage or from
a mirror: see the list of attacks below.

The fundamental issue at hand is that packages often contain
vulnerabilities. Most vulnerabilities eventually get fixed in later
versions, but end-to-end package signing provides no means for Bob to
determine whether a given package version includes fixes to all known
vulnerabilities, positive as he may be that the package really does
come from Alice. The solution to this problem is to maintain signed
package-sets, replicated across many machines.

Signed package sets and end-to-end package signing really are
complementary, as Duncan Coutts
[observed some months ago][coutts-sig-jan-email]. They work hand in
hand to ensure secure package distribution and system updates.

Chris Done has already tackled
[end-to-end package signing][chrisdone-signing] in a separate
proposal. This proposal is about signed package-sets.

[coutts-sig-jan-email]:
https://groups.google.com/d/msg/commercialhaskell/qEEJT2LDTMU/_uj0v5PbIA8J

[chrisdone-signing]: https://github.com/commercialhaskell/commercialhaskell/wiki/Package-signing-detailed-propsal

## Threat model

* Offline keys are safe and securely stored.
* Attackers can compromise at least one of the package distribution
  systemâ€™s online trusted keys.
* Attackers compromising multiple keys may do so at once or over
  a period of time.
* Attackers can respond to client requests (MITM or server
  compromise).
* Attackers know of vulnerabilities in historical versions of one or
  more packages, but not in any current version (protecting against
  zero-day exploits is emphatically out-of-scope).

An attacker is considered successful if they can cause a client to
build and install (or leave installed) something other than the most
up-to-date version of the software the client is updating. If the
attacker is preventing the installation of updates, they want clients
to not realize there is anything wrong.

## Attacks

* **Arbitrary package:** attackers should not be able to provide
  a package they created in place of a package a user wants to install
  (via MITM during package upload, package download, or server
  compromise).

* **Rollback attacks:** Attackers should not be able to trick clients into
  installing software that is older than that which the client
  previously knew to be available.

* **Indefinite freeze attacks:** Attackers should not be able to respond
  to client requests with the same, outdated metadata without the
  client being aware of the problem.

* **Endless data attacks:** Attackers should not be able to respond to
  client requests with huge amounts of data (extremely large files)
  that interfere with the client's system.

* **Slow retrieval attacks:** Attackers should not be able to prevent
  clients from being aware of interference with receiving updates by
  responding to client requests so slowly that automated updates never
  complete.

* **Extraneous dependencies attacks:** Attackers should not be able to
  cause clients to download or install software dependencies that are
  not the intended dependencies.

* **Mix-and-match attacks:** Attackers should not be able to trick clients
  into using a combination of metadata that never existed together on
  the repository at the same time.

* **Malicious repository mirrors:** should not be able to prevent
  updates from good mirrors.

* **Wrong author attack:** an attacker uploads a new version of
  a package for which he is not the real maintainer (assuming he is
  not a Hackage trustee either).

## Proposal

To avoid ambiguity, and in deference to established terminology in
other proposals, we'll call package-sets *snapshots*.

* A *package repository* is a single location containing any number of
  *snapshots*. A snapshot's content includes all package metadata,
  along with optional repository metadata, such as an authorization
  list (also called target delegations). Packages themselves are
  stored separately - cached locally or only on third-party servers
  (controlled by package authors).

* Each snapshot is identified by a hash of the content of the
  snapshot, including a link to zero or more parent snapshots, and
  a single cryptographic signature.

* The authorization list allows delegating trust from a single root
  entity for arbitrary subsets of the packages in any given snapshot.

* A package repository does not contain the packages themselves, only
  package metadata. The actual packages can be cached locally or
  served from any number of mirrors and/or developer specified
  original locations.

## Technical details

A package repository is implemented as a *Git repository*. A snapshot
is simply a *git commit*. It can be signed using Git's existing tools
for doing so: `git commit -S`. Commits are signed using GPG keys,
avoiding the need for duplicate identities separate from already well
established ones in common use. If in the future Git supports other
key types, then so will this proposal.

Clients are made to stop trusting a snapshot signature once
a configurable amount of time has elapsed. The recommendation in this
proposal would be 1 day, though paranoid clients may choose shorter
times. Upload services (i.e. Hackage) must ensure that a new snapshot
is created and signed at least once a day, otherwise clients will
start refusing to trust the latest snapshot.

### File layout

The file layout of each snapshot is as follows:

```
/packages/
/packages/preferred-versions
         /base/4.7.0.1/base.cabal
         /base/4.8.0.0/base.cabal
		 /base/...
		 /bytestring/...
		 /...
/snapshot-keys
/snapshot-keys.asc
```

That is, the `packages/` directory follows the exact same structure as
that expected by `cabal-install` today, if only for backwards
compatibility.

### Package metadata

The packages themselves are not checked into Git, only their `.cabal`
file or equivalent metadata. However, the certification chain rooted
at the git commit signature should extend to packages themselves, even
if end-to-end package signing partially obviates the need for this.
The reason is that end-to-end package signing makes packages
tamper-proof, but only for those packages that have end-to-end
signatures. Somehow including the hash of a package tarball in the
package metadata mitigates against MITM and server compromises so long
as the key used to sign a snapshot has not been compromised. The means
we choose to achieve this is by introducing a new field in `.cabal`
files:

```
package-hash: <SHA-256>
package-locations: <URL>...
```

where `<URL>` can be relative or absolute. Examples:

```
package-hash: 1060c8f4c97e3fa3e053f45e6f8972bcb31a4cfc
package-locations:
  /package/base-4.8.0.0/base-4.8.0.0.tar.gz
  http://example.org/packages/base-4.8.0.0.tar.gz
```

In this way, package tarballs can be hosted on a server of the
developer's choice, or Hackage, or both. Upon downloading a tarball,
end user tooling MUST check that the SHA-256 matches the hash embedded
in the name if it exists. Making optional the presence of a hash in
the name is for backward compatibility with the existing package
tarball scheme.

**Variations:** alternatives to consider here are having the hash
embedded in the name, such as

```
package-locations:
  /package/base-4.8.0.0/1060c8f4c97e3fa3e053f45e6f8972bcb31a4cfc.tar.gz
```

This eliminates the need for the `package-hash` field, and allows the
hash to vary according to the format of the package sdist (e.g. for
zip files). However, some hosts may be particular about the naming
scheme for files.

### The top-level keyring

The root of all trust in the update system emanates from a quorum of
root key-pairs. Each such private root key MUST be kept offline, in
separate, secure locations. We propose having 3 such keys, each
maintained by a Hackage trustee. All other top-level keys, in
particular snapshot keys, must be signed by at least 2 of the 3 keys
(alternatively: require 3 signatures always). Root keys rarely need be
used (about once a year), so it's easy to keep them more safely than
other keys.

Public root keys are distributed with end user tooling, such as
`cabal-install`. Users assign trust to these keys according to the
usual mechanisms for GPG keys, after collecting evidence about these
keys out-of-band. Revocation of these keys works similarly. Root keys
can be revoked via updates to `cabal-install` or otherwise.

Root keys are used to sign any number of snapshot keys. Any signature
of a snapshot key should expire after at most 1 year. End user tools
only trust a snapshot key if it has a valid root key signature.
Snapshot keys are less secure than root keys: they must be used every
time a new snapshot is created, so they are more difficult to keep in
a safe place. In fact some snapshot keys by necessity will be kept
online: one for each package upload service (i.e. Hackage, basically).

A *Hackage trustee* is an owner of a snapshot key signed by a quorum
of root keys.

Hackage itself, being an upload service, owns a snapshot key.

A snapshot can be signed using any key. However, end user tooling
should ignore a snapshot unless it was signed by a key whose ID is
listed in `/snapshot-keys` AND the signature of that file by root keys
can be verified AND the key has been signed by a quorum of root keys.
Snapshot key signatures can be obtained via the usual channels: key
servers. The reason for the existence of `/snapshot-keys` is to be
able to revoke snapshot keys in-band and synchronously, without
requiring that clients be required to poll a key server for revocation
certificates before each pull.

**Alternatives:** There are two additional designs possible here:

1. Do away with `/snapshot-keys` and have clients periodically poll
   key servers. This has two costs: a) increased latency before clients
   realize a snapshot key has been revoked, and b) increased
   complexity on the client side to schedule polling the key servers.

1. Replace `/snapshot-keys` with a `root.json` file, whose format
   would be as mandated by [Section 4.3][tuf-root-format] of the TUF
   framework specification. The `threshold` field for the `snapshot`
   role must be set to 1 (Git does not support multiple signatures per
   commit and in any case that would add little to no extra security
   for our use case).

[tuf-root-format]: https://github.com/theupdateframework/tuf/blob/db0ba6d0f1e1a86f066a89bfeeb469d729c9709a/docs/tuf-spec.txt#L485

### Naming snapshots

Clients are free to check out any snapshot that they wish with
a sufficiently recent snapshot signature (less than a day old). In
order to *find* snapshots, clients can use named references. The
`master` ref points to the latest "official" snapshot of the Haskell
open source community. Repositories are free to include refs pointing
to unofficial snapshots, for specialized purposes.

Tags can also be used to name historical snapshots.

## Dealing with snapshot key compromise

We expect end user tooling, such as `cabal-install`, to ship with
a limited set of root keys. These keys are used to delegate trust to
any number of snapshot keys. Anyone with sufficient access rights can
create a new snapshot in a package repository, but unless a snapshot
has been signed using a key listed in a signed `/snapshot-keys` file,
end user tools will typically ignore the snapshot, or display
a warning to the user about potential vulnerabilities in that
snapshot.

## Analysis

* **Arbitrary package:** impossible without compromising a snapshot
  key, and even then only for packages that don't have end-to-end
  package signing. And only until compromised key gets revoked and
  a snapshot signature expires.

* **Rollback attacks:** thanks to Git's cryptographic guarantees
  derived from making the identity of a snapshot depend in part on the
  identity of the parents, rollback attacks are impossible without an
  up-to-date client noticing. Rollbacks are only possible up to the
  latest pull, and even then, only within the limit of 1 day's worth
  of updates at most. Furthermore, any attempt at a roll back attack
  on many clients at once is very likely to raise alarms immediately,
  because chances are at least one such client is up-to-date, and
  therefore will notice the non fast-forward ref update that would
  result.

* **Indefinite freeze attacks:** impossible without compromising
  a snapshot key, and only until clients notice key has been revoked.

* **Endless data attacks:** possible. Can be mitigated by introducing
  a small timestamp file fetched via HTTP before the `git pull`, but
  this type of attack is not very relevant for our use case and are
  easy to mitigate via tooling. It's not very relevant because no
  server should ever be running `cabal update` as a cron job and then
  auto rebuilding software in an unsupervised fashion.

* **Slow retrieval attacks:** same as above.

* **Extraneous dependencies attacks:** impossible without compromising
  a snapshot key.

* **Mix-and-match attacks:** impossible without compromising
  a snapshot key.

* **Malicious repository mirrors:** impossible without compromising
  a snapshot key and even then, only until clients notice snapshot key
  has been revoked. Freeze attack possible until snapshot signature
  times out, i.e. for at most 1 day. Indefinite freeze attack not
  possible unless snapshot key compromised.

* **Wrong author attack:** possible if attacker can somehow thwart
  Hackage authentication. Possible if attacker compromises a snapshot
  key. In both cases, only possible for packages that do not have
  end-to-end singing.

In sum, this proposal thwarts all types of MITM attacks, even for
unsigned package content, prevents malicious repository mirrors from
mounting most attacks (and mitigates freeze attacks to be limited in
time), institutes an easily inspectable audit trail for detecting and
reversing any effects of a successful server compromise, and severely
restricts the scope of any rollback attack even when a snapshot key is
compromised.

One can reasonably assume that if Hackage is compromised, then at
least one a snapshot key will be compromised. So breach of Hackage is
still bad news - there's no silver bullet. Further, a breach of
Hackage will allow an attacker to inject arbitrary packages. The only
way to mitigate this is to put in place end-to-end package signing,
though even then replay and indefinite freeze attacks will still be
possible.

### Why Git?

Git gives us a lot of mechanism for free. Key to preventing rollback
and mix-and-match attacks is making sure that the signature of any
future snapshots depends on current history. This is what Git does
well. In fact, it's uncanny just how good the fit is between the
requirements for preventing rollback attacks and the features of Git.

Second, Git provides us with a robust and efficient update delivery
mechanism. Clients don't need to download the full history (not even
to check signatures), and when a new snapshot is added, distribution
of this new snapshot is done efficiently, with little information
required from the client.

### Why aren't package tarballs included in repositories?

Because importing all of Hackage into Git, including package content,
today makes for repo of size 1GB, which when checked out is more than
a dozen gigabytes. While many might argue that disk is cheap,
bandwidth is not, depending on your locale. Not all countries have
internet connectivity as good as Korea and Norway. Especially in
developing countries, it would be a real liability for Haskell if the
first step before doing anything is having to download a 1GB Git
archive. Especially considering that given the current growth curve,
the Git repository with *all* content imported will likely be hitting
2GB by this time next year, and so on.

Yet, it's true that importing all package content yields a suprisingly
small Git repository. But that's partly due to the fact that no one
has uploaded on Hackage a package of the size of Libre Office (yet!).
Developers should feel free to include whatever large assets they wish
in their packages, now or in the future, without fear of *forever*
bloating the package repository that people around the world use to
write Haskell.

Finally, package content need not be hosted on Hackage at all. Hackage
is a central authority for tracking what open source packages are out
there. All that is needed for that purpose it to track package
metadata, along with information about where to find the package
itself. The central package repository that everyone uses should
include the *minimum* information necessary to get the job done.

Some have argued for the ability to easily mirror the entire package
repository and package files locally, aka offline mode. Supporting
offline work is important, but arguably this is a tooling problem: end
user tools can easily enough grab all current package files for later
installation. Alternatively, a mirror with package content included
can be provided.

### Comparison to TUF

[TUF](http://www.theupdateframework.org) is a general framework for
software update, which has been proposed but not yet accepted in the
Python and Ruby communities (and proposals in both communities have
seemingly stalled by this point). TUF is generic enough that **this
proposal can be seen as one particular instance of the TUF
framework**. Yet we leverage Git to good effect, to achieve more
robust snapshot signing than is proposed in the TUF specification.

TUF defines a number of "roles": root, snapshot, timestamp, targets,
mirrors. Like TUF, we have a "root" and "snapshot" roles. The
"mirrors" role is optional and not useful for our use case. Following
the Python and Ruby TUF proposals, we use the same key for the
"snapshot" and "timestamp" roles, so that effectively there is no
practical distinction between the two roles. The targets role is
covered by end-to-end package signing, and is orthogonal to this
proposal.

The TUF framework ignores Git entirely, yet seeks to emulate (some
might say *reinvent*) some of its key mechanims. In particular, TUF
signs a top-level `snapshot.json` file for each snapshot. In order to
protect against rollback attacks, this `snapshot.json` file includes
a version field that MUST be auto incremented at each new snapshot.
This means the history of snapshots must be linear. It's also easier
to get wrong than just creating a new Git commit.

Another example of complexity appearing in the TUF spec, which simply
disapppears when using Git, is Section 7, which includes best
practices for creating "consistent snapshots", so that clients can
still download a snapshot while a new one is being created. In this
proposal, we get the equivalent of all of Section 7 for free, by
virtue of reusing Git. Likewise target file update, which is much of
what Section 5.1 is about, becomes simpler. We don't have to compute
ourselves what package metadata files have been updated and reinvent
incremental transfers: Git does that for us.

Finally, the TUF spec eschews GPG keys. This strategy quickly becomes
a liability. In case of server compromise, TUF is ill equipped to deal
with cleaning up the ensuing mess. Because snapshots are not linked
together into a history, identifying precisely what changes an
attacker made becomes difficult. Administrators have no choice but to
forcibly restore an old snapshot from cold storage, which could be
days or weeks old, and hence pulverize recent package uploads by
developers. Closer to the point about GPG keys, in our proposal,
clients can discover out-of-band by querying multiple key servers if
necessary that the trust for some snapshot key has been revoked. In
TUF, clients can only rely on in-band signaling, which is problematic
when a mirror server has been compromised and an attacker is mounting
a freeze attack.

## Task plan

Effort estimate: 6 days.

* Teach hackage-server to push new git commits upon package upload.
* Teach hackage-server to sign git commits using configured snapshot
key.
* Create Hackage root keys. Create Hackage snapshot key. Have snapshot
key signed by root keys.
* Define procedure for safe storage of root keys (likely reuse an
  existing one).
* Add optional additional backend to `cabal-install`: pull package
  metadata via Git.
