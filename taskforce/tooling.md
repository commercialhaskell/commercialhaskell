## Haskell Tooling Task Force

__Started__: March 23, 2015

__Goal__: Improve the situation in Haskell build tooling. This was spurred by
an email titled [Wither the
Platform](https://mail.haskell.org/pipermail/ghc-devs/2015-March/008567.html)
about the current state of Haskell Platform. Michael Snoyman [called for the
task force](https://mail.haskell.org/pipermail/ghc-devs/2015-March/008586.html)
in that thread. We should start by:

* Identifying problems faced by users
* Assess current solutions
* Find common ground between solutions
* Make plans on how to unify into more holistic tooling

__Members__:

* Anthony Cowley
* Chris Done
* Michael Snoyman
* Michał J. Gajda
* Miëtek Bak

__Problems__:

* Consistent means of installing GHC, cabal-install, etc
* Avoidance of "cabal hell"
* Allow new users to get started easily
* Provide power for more advanced users
* Get versions of packages with bug fixes
* Minimize installation/compilation time

__Solutions__:

* [Getting Set Up](https://github.com/bitemyapp/learnhaskell/blob/master/README.md#getting-set-up)
* [The Haskell Tool Stack](https://github.com/commercialhaskell/stack)
* Halcyon
* Haskell Platform
* LTS Haskell
* MinGHC
* Nix
* Stackage Nightly
* Ubuntu PPAs for GHC and cabal-install
