# Haskell at Front Row

## The mission

Front Row Education was founded to change the way math education is done in a modern day classroom. In the web universe
we have all sorts of great tools for tracking, analyzing and incentivising user behavior: complex analytics, rich data
visualizations, a/b testing, studying usage patterns over time, cohort analysis, gamification etc. We figured: instead
of using the above to have granny click on more ads, let's make these powerful techniques available to teachers,
parents and school administrators to make math education more engaging and effective.

Front Row allows schools to track student progress over time, identify areas of struggle, learn how to address them, all
the while encouraging more quality practice. Learning math this way becomes a interactive and compelling experience,
providing immediate feedback and adjusting content with every answer. As students practice, they generate rich data that
school staff uses to continuously course-correct and fill in the gaps.

Numerous experiments from past years show that making Front Row a regular part of a math classroom leads to
improved conceptual understanding, a lower rate of students falling behind, and improved scores on state
tests. As of today Front Row helps over a million students in their regular math practice, and has been used in over 30%
of US K-8 schools.

### Our journey to Haskell

As of today Front Row uses Haskell for anything that needs to run on a server machine that is more complex than a 20
line ruby script. This includes most web services, cron-driven mailers, command-line support tools, applications for
processing and validating content created by our teachers and more. We've been using Haskell actively in production
since 2014.

At the time of the switch we were already familiar with the functional programming world. The central piece to the Front
Row system is the JSON API used by both the student and teacher web experiences. I wrote the first version of the API in
2013 in Clojure on top of the Ring/Compojure micro-framework. At the time I didn't have plans for the API to grow to
serve the kind of size and traffic we see today: it was mostly a way for me to really dive into functional programming
and understanding design challenges that other popular frameworks had to come across.

Building your own framework is a fantastic learning experience, but it is also a significant commitment: without
investing a ton of time and effort into the framework, you'll end up with something very bare-bones and hard to turn it
into a production quality, fully-featured application. It takes innumerable iterations to make a framework extensible,
modular and well maintained with a team of 1-3 developers, busy with dozens other tasks that a fast-moving startup
demands.

Clojure at the time didn't offer any alternatives as far as web frameworks were concerned, and we were already starting
to see the inherent critical weakness behind building large modular systems in dynamically typed languages: refactoring
is a serious pain and something you will avoid at all costs because it's hard to ensure you're not breaking anything.
It's not that bad if you have ONE codebase that doesn't have dependencies, but once you get into two digits you're in
for a bad time.

Switching to Haskell and the Yesod framework seemed like a natural step forward: a strongly typed, purely functional,
highly expressive language that would finally allow refactoring and moving fast to be painless. On top of it, a
beautifully designed, extensible web framework with years of polish, one of the best high-performance web servers in the
industry, extreme attention to type safety, and an all-star team of OSS contributors supporting it.

Moving from Clojure to Haskell didn't feel like a massive jump: a lot of concepts translate pretty closely, although
Haskell offers a much richer vocabulary than just maps and vecs. Monads, type classes, IO etc. eventually clicked, and
it was smooth sailing after that.

## Advantages of using Haskell

Where does Haskell fit into all of this you say? As the development team of a small early stage edtech startup, we have
two main goals:

1. Iterate as fast as possible on new educational concepts, business model experiments and user feedback. Basically,
   crank out as much code as possible while keeping the quality bar very high.
2. Stretch our runway, be conservative with our very limited resources

Haskell fits in pretty well with both of goals.

### Static typing

First of all, static typing is essential when it comes to keeping the system always in a working state. Coming from a
dynamically typed universe, it's surprising how much time you can save on writing unit tests, because you are getting
more certainty from the compiler: no more null exceptions, no type mismatches in function calls, no more forgetting
about dealing with the empty list case etc. A whole class of pesky, incredibly common and banal bugs is eliminated from
your work: you now have more bandwidth to worry about implementing user stories instead of obsessing that your
application doesn't blow up due to sloppy oversight.

I still remember one of my biggest Haskell/Yesod "aha" moments: not only does Yesod make sure that routes in your HTML
are type-safe, but even image files linked in <img> tags are verified to exist on disk by the compiler. No .jpg, no
build, it's that simple. It's a level of guarantee that dramatically increases your confidence in the code at barely
any cost.

### Modularity

Modularity is another big one. We have a central module at the bottom of every one of our web applications, APIs,
tools and cron binaries. This module wraps the database entities and the SQL logic necessary to access them. It also
provides a lot of common shared functionality that should not be implemented more than once. Since the schema changes
very aggressively, we need a way to make sure our applications are updated ASAP, we can't wait for things to blow up in
production. Updating our entity definitions in that one module prevents every application built on top of it from
compiling again until the change is dealt with.

No more API call mismatches, no more using an old schema, no more apps running against an old deprecated version that
can lead to breaking the db state. As many others have stated, Haskell is the first language out there that feels like
it manages to achieve true modularity: purity and defining what context a function is allowed to run in ensure that a
library call can lead to no surprises. Testing side-effect free functions is much simpler than continuously dealing
with system state.

### Efficiency

Regarding the second point, why would Haskell stretch your runway? Simple. You're writing fewer bugs, you're
reusing more code, new developers are causing less damage, and you have more room to deal with technical debt before it
bites you. Purity and static types allow a team to aggressively refactor the codebase without having to worry that they
might have forgotten to update something: a combination of a light layer of spec-style tests and a very picky compiler
provide you with most of what you need to make refactoring a non-issue. More refactoring = more long-term productivity,
higher team morale, more pride in one's work. Doing the same with a Ruby is as fun as pulling teeth.

All of the above adds up to needing fewer developers, as less time is spent on maintenance, which ultimately equals a
higher chance of your company getting somewhere thanks to the more frequent iterations. The more stuff you try, the more
likely you are to find or expand that business mechanic that will carry your business forward.

## Trouble in paradise

This is not to say that things aren't all perfect though, and there's still plenty of room for improvement in the
ecosystem.

### Building

Build times, especially once the whole constellation of Yesod and Persistent packages are brought into the mix, are not
insignificant. It still takes a good 5-10 min to build our larger web application on our beefiest machines. There are
optimizations that can be made in this space which we haven't adopted yet, such as caching already build object files to
avoid having to re-compile them every time, so I'm confident this will be a non-issue in the nearby future, but it's
still worth being aware of. GHC works hard, you need to provide it with enough juice or time to let it do its job.

### Testing

The testing frameworks out there are still fairly spartan from the developer experience standpoint. If you test Yesod
with hspec, the premier BDD library for Haskell, there's currently no way to insert a bunch of rows into the database
during fixtures and pass the results into the individual test cases. You have to wrap each test case in additional
function calls to pull that off, adding more boilerplate to your tests.

Additionally, it's not possible to find out which one of your specific test cases failed when checking for multiple
conditions within the same "it" block. This means that if you need to check the state of the system after an HTTP
request, you have no clue which one of the checks failed.

Fortunately the developer(s) behind these libraries are responsive and happy to look into improvements. At the very
least they're glad to point other developers in the right direction towards a PR.

This has in general been my experience with the Haskell community: things aren't perfect, but folks are always looking
for a way to improve the ecosystem and want Haskell to be the best language to develop in. People are trying to carve
out their little slice of paradise, and are willing to put in the hard work to make it happen.

### Docs

Documentation is still not quite there and the initial onboarding of new developers is still rough. There are only so
many snippets to Google for, compared to e.g. Ruby and Python. A lot of documentation is very barebones and requires
diving straight into the source, which is fine for a proficient Haskeller, but not for an already terrified beginner.

Many times I've witnessed senior developers get very frustrated when something wouldn't compile for hours and they
couldn't find any help to move forward: be prepared to assist them before they get too grumpy. Some projects are better
about it than other: Yesod and Persistent have extensive documentation and the FPComplete crew have numerous tutorials
out there to help. New books come out once in a while with fresher snippets: the time-tested *Real World Haskell* is now
fairly outdated, but the more recent *Beginning Haskell* is perfectly relevant. Many channels on IRC are available:
 #haskell-beginners, #haskell and #yesod, although sometimes it can take work to get the answer you're looking for.  More
than once I heard the comment that documentation seems to be written by wizards for other wizards, and if you're a lowly
initiate, you will have a rough time.

I've personally had the privilege to help all of our developers skill up in Haskell and Yesod, and I've become a huge
believer in the power of having someone more experienced guide you along the way. What took me several months of
learning, mostly by myself, now takes our developers a couple of weeks of quality coaching. It took me a while to grok
monads, type classes, type families etc., however, properly guided developers can figure it out in a matter of hours.
Having a good teacher on your team will speed adoption within the organization immensely.

### Strength in numbers

We once experienced a very frustrating issue that got us thinking about our full commitment to Haskell as a company.

When we switched our main API to Yesod (a full rewrite), we almost immediately ran into the issue the API would burn up
close to 95% of available CPU on whatever AWS EC2 instance it was hosted on. We upgraded machines, just to see if we
could cheat our way out of fixing this by throwing money at the problem, and even with a $600/mo 16 core box, the API
still managed to flood all of the available cores with barely any traffic hitting it. I personally spent a good week
banging my head against it: was it resource contention? Was it a really big oversight in one of my handlers? Was it
misconfiguration?  Was it something about the EC2 environment? Why doesn't this reproducing AT ALL under profiling? Was
it our database connection pooling? I threw a lot of screenshots and code samples at the community both on Google Groups
and IRC: nobody else had ever seen anything like it. Uh oh.. All the while customer support requests are pouring in,
teachers are aggravated, the team is looking at the devs and "their latest shiny toy", tapping their collective foot.

This is the part where picking exotic tooling for your stack can be a dangerous beast: "given enough eyeballs, all bugs
are shallow", and when only a dozen teams out there are using your libraries at your scale, you are on your own when it
comes to fixing issues. With Rails, there's enough volume of developers that there will be enough projects of every
scale to burn-in your tool of choice. That's simply not the case with Haskell's usage numbers.

What this means is that if you're planning to bet the farm on Haskell, you need to be ready and comfortable with the
idea that you might have to get your hands dirty, might be the first person to figure out a solution to the problem
you're seeing. This requirement is pretty much non-existent in .NET / ruby / python at al. Start small, start simple,
let the tooling grow on you as you gain experience. Start with tools that aren't mission critical until you're more
confident.

### However..

It bears mentioning that the above concerns are being actively addressed by the community and the state of things is
rapidly improving:

* Cabal, the Haskell package manager, was a real pain to work with just a few years ago and "cabal hell" is still part
of Haskell vernacular. However, with sandboxes and consistent version snapshots provided by FPComplete as Stackage LTS,
that problem has been mostly resolved.
* Build times are slow, but the community is coming up with improvements such as halcyon that should alleviate things
considerably.
* Docs have gotten dramatically better over the past couple of years. There's been a big push towards keeping fresh,
community-maintained, easy-to-follow and beginner-friendly instructions such as those provided by [Chris Allen's Learn
Haskell](https://github.com/bitemyapp/learnhaskell). We now even have IRC channels tailored specifically for beginners,
e.g. #haskell-beginners . Today newcomers become more productive much faster than they did a few years ago.
* The community has been recently doing a better job at outreach and we've seen many new developers come make Haskell a
permanent part of their toolbox. With more participants, tools get more fully-featured and more maintained.

## Conclusion

It's a very exciting time in the history of computing to jump on the Haskell train. Yes, the community is tiny and one
might get little hand-holding compared to more popular ecosystems, however Haskell offers obvious benefits to software
teams who can power through the initial pain period.

Today Haskell offers some of the best tools around for delivering quality software quickly and reliably, minimizing
maintenance cost while maximizing developer enjoyment. To me Haskell is that dream of "developer happiness" that we were
promised many years ago by the Ruby community: I can write beautiful, short, expressive and readable code that will
perform phenomenally and stand the test of time and continuous change. What more can I ask for?
