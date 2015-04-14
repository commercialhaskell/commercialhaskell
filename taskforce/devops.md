## Haskell for Devops

This page is for collecting information and collaborating on the general idea
that __Haskell is a great tool for devops__. The [original mailing list
thread](https://groups.google.com/d/msg/commercialhaskell/L8Sca1X0QDA/3qU64WKbu-wJ)
covers a lot of information, but hopefully the following points will be a good
summary:

*   Devops is a broad term that can mean many different things. We're going to adopt the following definition provided by Arnaud Bailly:

    > The essence of DevOps as I think Patrick Debois has explained it for
    > years is exactly to be sure there is NO separation between the two areas
    > you are mentioning, viewing the Development to Operations as a continuum.

    This covers things from config management, to CI systems, to web services.

*   Haskell's ability to write low-runtime-dependency, high performance
    applications using its wide array of composable libraries makes it
    appealing for putting together various devops services.

*   Haskell plays nicely with existing devops tools like Docker and Ansible, as
    demonstrated by how people in the Commercial Haskell Special Interest Group
    are using Haskell.

Please feel free to expand on this.

### Projects for collaboration

To be filled in

### Blog posts

As people in the SIG write up information, please link to it from here

*   [Warp in a tweet](https://twitter.com/snoyberg/status/587909384510042112). Run a high performance static file server with a single line:

        docker run -it --rm -p 3000:3000 -v /var/www/html:/var/www/html:ro snoyberg/warp /usr/bin/warp -d /var/www/html

#### Ideas for future content

* Propellor
* Shake
* Bouncy
* wai-crowd

### Projects

* [haskell-scratch](https://registry.hub.docker.com/u/snoyberg/haskell-scratch/), a minimal Docker image containing some necessary shared libraries for shipping GHC-compiled applications
