SCM integration has the potential to

  * provide a direct and secure package transport mechanism
  * add additional security to signed packages
  * add end-to-end security to unsigned packages
  * make local forks practical to deal with in a secure way and easier to install

This ends up having a lot in common with the package signing proposal in terms of decentralization, but also some orthogonal and complementary capabilities.

One way to think about package distribution is by asking: what happens if we get rid of a package intermediary like Hackage?


# Completely De-centralized

I search for a package on google, find its Github page, and look at the releases.
I can use git to securely checkout the repo at the release tag and vendor the sources.
Unfortunately I need to manually resolve dependencies and version issues.


# Secure transport

But lets note that I have accomplished secure transport via git.
Other programming language installers such as `go get` and Ruby's `Gemfile` have SCM integration.
When you use `go get`, it will demand that you install `git` or `hg`, etc, so the user is responsible for having a tool installed that accomplishes secure transport.


# An essential workflow

The SCM workflow is already used today for local forks, except that we often find the SCM url on hackage and may have problems determining the SCM commit of the release version. This situation cannot change under the existing proposals because there is no guarantee that there will be a commit matching the tarball. So the only secure local workflow under the signing proposals is to not clone from the SCM but to unpack a tarball. This creates a lot of extra overhead to contribute back to the original repository.

# A trusted workflow

When an author puts the url of the package on Hackage it is an implicit statement of trust in Github knowing that users may create a local fork from it. The package author already trusts Github for their normal development workflow of pushing their repository. It is not completely blind trust: the distributed nature of git means there are some checks against tampering and the collaborative nature of github means there is visibility into some forms of third party attacks.


# Bring back centralization!

We have some big problems with finding and resolving dependencies in the SCM workflow.
So we need central locations where package information can be published.
The published SCM information (in addition to what is required in a cabal file) is

  * URI: location to fetch package source
  * SCM  (git, mercurial, darcs): how to fetch package and move to the commit
  * release commit: combine with URI to obtain the source code
  * Optional: subdirectory to find package within the repo


# Signing proposal weaknesses

## Security

### Compromised private key

Done properly signing leaves one major attack point: the private key of the key pair. Adding the SCM attributes to the package data makes attacks much harder. We can verify the package contents against what is in the SCM. The attacker must also compromise something at the package SCM url. In the case of Github, Github is unlikely to be completely compromised. A more likely scenario is for an attacker to gain privilege to commit to a repo. In that case, the attacking commit will be very visibile.

If a user's key is compromised than their SSH key may well be also. But revoking a PGP key is only possible if the user has their revokation key, whereas SSH keys are always revokable.

### Visibility of an attack

Github has a lot of visibility. But if the metadata stores can probably email the package authors when uploads occur.

### Imitation and the Web of Trust

The SCM and Github can help validate the web of trust that is going on.

## Usage or lack thereof

What if package authors don't want to bother signing their package? They don't want to install PGP, but they most certainly use a SCM. Assuming we can auto-detect the SCM, The sdist step is now just:

    cabal sdist v0.4.3

The command will verify that the tag exists in the local repo and in the project URI specified in the SCM information.

If we can get the author to securely log in to Hackage and enter the project URI, things become more difficult to hack because we can verify information against what is in the SCM. If the SCM or package directory information is changed, the SCM fetch is not going to work. Changing the SCM commit and the cabal version allows an attacker only the easy ability to attempt to convince us to use an older version of the package, not one that they create (because we can verify that against what is in the SCM).

So we need a secure client to to tell Hackage about a URI. We all have one: a browser. Creating a new package means logging in, writing down a package name, a SCM, and a URI. Releasing a package is a later event that requires giving Hackage a cabal version and a SCM commit revision. Even if this meta-data release is insecure we can check the data against the SCM.


# Security weaknesses

A big weakness of the SCM approach is that we must trust the URI. If Github is hacked, we are hacked. SCM mirrors, a sha of package contents, and the difficulty of faking a git sha can prevent tampering with *previous* versions. Publishing *new* versions from a compromised SCM repo can be protected against by

  * authenticating the sending of giving a cabal version and a commit revision
  * specifying multiple SCM urls that must all be pushed to (this will require every url to be compromised with write access).


## Centralization

Another weakness is the centralized distribution of package information: Hackage could be compromised. We can solve this by having multiple Hackages. But I don't really mean Hackage, I just mean a simple meta-data repository with API access. These servers can be running in multiple locations. When you release a package, you push your information to multiple servers (potentially insecurely, although securely giving the URI for a package would be required). When you download your package, you check package information from multiple servers and verify that they are all the same.


# Distribution

The signing proposals assumes centralized package distribution (with mirroring). The SCM provides a secure direct transport mechanism. However, we are relying on Github being up.


## Local SCM verification

A company can operate a local Hackage by downloading the package meta-data and cloning all the SCM repos (that it uses). This local Hackage could be a server that polls for updated meta-data and gathers the latest SCM updates. A persistent update process helps avoid SCM downtime issues. In addition, this process would automatically verify that the SCM contents match the meta-data.


# SCM benefits unrelated to security

Cabal already has a command to fetch the source. If it also knew how to checkout a specific release, that helps to hack on an open source library you are depending on.

There is a problem with having a local copy of an open source library though. You should bump versions in your local copy, but that will just confuse cabal because it also finds version info on Hackage. I think there is an opportunity here to integrate SCM commit hash information to avoid confusion.

# Conclusion

Adding package signing is the clearest way to secure the basic package installation workflow, if we can get everyone to do it. SCM integration can complement package signing security.

SCM integration has the potential to add real end-to-end security by itself. However, the security has weaknesses unless a user can securely transmit some release metadata or change their workflow to push code to multiple locations.

SCM integration also offers

  * local forks that are practical to deal with in a secure way and easier to install
  * a direct package transport mechanism that is secure
