Purpose 
====================

The Haskell community already has (and produces) many forms of
documentation:

-   Auto-generated type documentation for all libraries (a feature
    of static typing)
-   API reference documentation (Haddocks)
-   Library tutorials
-   A number of books (Real World Haskell, Learn You a Haskell,
    etc)
-   Academic papers

Each of these forms of documentation are necessary and vital
components of the Haskell community, and we should do everything we can
to continue to produce and improve them. This document would like to identify one area that is currently unmet, and propose a way to solve it:

__Intermediate level, guided tutorials__

One of my coworkers at FP Complete- Sal Daoud- will be making a *different*
proposal for other missing documentation, but that is not included here.

Intermediate level, guided tutorials 
=================================================

(For those familiar with it, this idea is not dissimilar from my
previous [Mezzo
Haskell](https://github.com/fpco/mezzohaskell) initiative.)

Imagine a developer who has no knowledge of Haskell. We have some
very easy starting points: read RWH, real LYAH, look at some
introductory tutorials on School of Haskell, etc. At the end of these, a
developer will typically have a pretty strong grasp of the basics of
Haskell.

Unfortunately, “leveling up” from there is not so rosy a story.
Those of us who have done so have typically done so over many months and
years of picking up new knowledge. To demonstrate the problem, let me
give two examples:

-   The developer hears about monads. “Oh, I’ll just search for a
    monad tutorial.” Soon enough, he/she is studying nuclear physics to
    understand why an astronaut would need to eat a burrito after crash
    landing in the Pacific.
-   The developer wants to implement an algorithm that is best
    done with a temporary mutable array. Haskell intermediates and
    experts would probably say “use the vector package in the ST monad.”
    But none of the introductory texts (to my knowledge) give any hint
    at this. Thus, our developer won’t even know the right terms to
    search for.

The first problem is one of *quality* of documentation. The second is one of
*discoverability*. There is also a
third problem: *existence* of
documentation. For some topics, no real tutorial exists. Vector, for
example, does not have any such tutorial. It’s not a particularly
difficult package to use, but the lack of a clear step-by-step usage
guide certainly makes its usage harder for beginners.

One possible solution to this is to get someone to write the
intermediate Haskell book. There are two issues with that
approach:

-   If no one has done so yet, that implies that the people with
    the knowledge to do so don’t have the time/energy/inclination/etc to
    do so. Complaining about it on the Commercial Haskell mailing list likely won’t
    change anything.
-   There is a very broad category of topics to be covered, and
    its unlikely that any one person is an expert in all these
    topics.

I’m going to propose that we have a combination of *open editing* and
*curation* to address all issues on the
table.

Proposal 
---------------------

The Commercial Haskell group (aka the SIG) will maintain a new repository
(haskell-docs?). This will be a repository of
Markdown files (since it’s a generally used and understood format
already). Commit rights to this repository will be granting very
liberally. The idea will be that individual files will have a main
author, but others are very much encouraged to make minor improvements
to these files (e.g., don’t send a pull request for a grammar fix, just
do it). More major changes, however, should be made via pull request or
issues.

On top of this, we will add a level of *curation*. There can be multiple curators, each
with different goals if desired. I will speak on the assumption of a
single curator, focusing on the topic of “intermediate Haskell
development.” It will be this person’s job to:

-   Create an outline/table of contents for “intermediate Haskell
    development” (I’ve worked on this quite a bit for Mezzo Haskell, and
    would recommend that be the basis for such a ToC)
-   Advertise this ToC and encourage people to write content for
    it
-   Include (via linking to the Markdown files) content in the
    repo, or even external content such as random blog posts
-   In cases where content is not up to the curator’s standards,
    work with authors to improve it

The SIG would be a great place to discuss and coordinate
documentation writing. And if the “intermediate Haskell development”
outline is too narrowly defined for some people’s tastes, there would be
nothing stopping them from creating their own, separate outline.



I’m notoriously bad at licensing issues, but we should have a
standard license under which all content added to the repo be licensed.
I’ll offer as a strawman the [Attribution 4.0
International (CC BY 4.0)](http://creativecommons.org/licenses/by/4.0/) license.

Possible improvements 
----------------------------------

By using an open system of Git +
Markdown, I’m hoping we could have many layers
of awesomeness added on top of this system. Examples:

-   FP Complete could add support to School of Haskell to display
    these contents, complete with active Haskell snippets
-   Someone could easily generate a PDF/ePub and distribute it as
    a book
-   Theoretically, we could even make a print edition of the
    book
-   A web-based display that allows for per-paragraph commenting
    (likely integrated with School of Haskell)
