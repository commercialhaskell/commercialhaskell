Being a maintainer of an open source package inherently involves some level of
communication. This communication can both be *outgoing* (e.g., announcements,
tutorials, documentation) and *incoming* (receiving bug reports, feature
requests, and patches). This document is intended to give some guidelines on
how to do this in the most effective way possible.

I should make it clear that this is an incredibly opinionated article, and
unashamedly so. Many people like to manage their projects in different ways.
Additionally, the goals that I'll be setting out below may not be universally
shared, though they are my personal goals, and I believe the goals of many
members of the Haskell Commercial SIG.

All of that is another way of saying: you're already doing a great service the
community by providing your software in the first place. Do not feel that this
document is trying to force you to do anything you don't want to. It's advice;
take it or leave it. (Or, if you're so inclined, suggest improvements.)

This document is also a work in progress, and does not cover all topics implied
above.

## Incoming communications

Supposing you've written some good piece of software, ultimately someone is
going to contact you about it. This comes in many forms:

* A bug report
* A feature request
* Patches (usually to address one of the above two issues).

I'll generally assume projects are hosted on Github, and therefore use
appropriate terminology (issue, commit and pull request), though the ideas
translate directly to any place code is hosted.

### Goals

You are intimately familiar with your code base. You likely have plans- whether
documented or not- of where you want the project to head. You may know about
reasons that certain features should or shouldn't be included. In other words:
no one in the world is more knowledgeable about your project than you are!

It's therefore important to remember that someone coming from the outside won't
have all of that background knowledge. The goal of the communication process is
to give this outsider as much information as necessary to let him/her
understand what's going on and participate, without inundating them with too
much content to possibly digest.

### Recommendations

__Acknowledge receipt.__ If someone sends a pull request and doesn't hear
anything back for two weeks, it can be disheartening. Silence can easily be
taken as disinterest, lack of maintenance, or even (by some extreme people) a
personal insult.

That's not to say that you are a slave to requesters. There are many valid
reasons for you to not immediately (or ever) act on a request, such as:

* You're incredibly busy at the moment
* You can't reproduce a bug
* You don't understand the report
* You can't test a pull request (e.g., requires an OS you don't have access to)

In all of these cases, the important thing is to *be clear with the requester*.
A note along the lines of "I'm traveling for the next 3 weeks, I will look into
this after I get back," is incredibly helpful. You can be even more vague: "I
have a looming deadline at work and can't look into this at the moment, I will
return to it when I have time."

In cases where you have said that you will return to an issue, it's important
to actually remember to do so. Set a reminder for yourself with your favorite
organization program (todo list organizer, Emacs org mode, etc).

It's difficult to give a hard deadline on how quickly you should acknowledge
receipt. I think under 24 hours is the highest level of responsive to be
expected of an open source developer. I'd say two weeks is very much on the
long side. Pick something you feel comfortable with and try to stick to it. If
there will be exceptions (e.g., you'll be hiking through the rain forest for
two months), make a public announcement. Even better: try to find someone to
co-maintain your projects during that time.

__Have a clear status for all incoming issues/pull requests.__ This is very
closely related to the previous point. Someone coming from the outside can be
frustrated by lack of clear status of an issue. Here are some theoretical
states any issue may be in:

* Awaiting initial review by maintainer (which, as mentioned above, can be confused with "ignored," so try to keep this time period short)
* Acknowledged, being reviewed
* More information needed from the requester
* Some disagreement about whether this should be accepted into the project
* Acknowledged, but maintainer doesn't have time/inclination/ability to implement him/herself

The most important thing here is to __make this status clearly known__. As a
maintainer, you could probably sit down with someone, go through your entire
issue tracker listing, and say exactly what the status of each issue is. The
goal though is for someone *completely disconnected from the project* to be
able to do so! Try to look at things with unbiased eyes and see if that's the
case.

__Document side communication/have a canonical place for discussion.__ This is
probably where the previous point gets tripped up a lot. Imagine you're
discussing an issue on Github with someone, and then you catch each other on
IRC and discuss it for the next two hours. You come to the conclusion that,
e.g., the bug report is valid, and that the other person will work on a bug
fix. Wonderful, that's real progress!

But does the original requester know about this? Odds are no. It doesn't take
long to document this, a quick comment in the issue itself along the lines of
the following will suffice:

> I discussed this with Alice, and she said that Bob had a similar issue. Alice
> is going to follow up with Charlie to get a bug fix written.

A side benefit of all this: when you come back to look at things 3 months
later, it's a lot easier to remember what happened this way.

__Have a single person driving the issue.__ If you have multiple maintainers on
a project, it's all too easy for issues to be lost in the void between people.
When something new comes in, it's a good idea to quickly identify who's in
charge of the issue. Having one person be in charge of triaging issues can be a
good approach. This can include having some kind of a watchdog timer: if no one
else takes responsibility for this in X amount of time, then person Y will
either take over or ask someone to take over.

__Have a clear decision making process in place.__ This most commonly comes up
with multi-maintainer situations, and therefore it's related to the previous
point. However, it *is* a separate issue. There should be a clear process in
place for making decisions. The default process that most everyone will assume
is "maintainer will make a decision." But this may not always be the case. As a
trivial example, with core libraries in the Haskell ecosystem, any significant
changes will likely be discussed with the libraries mailing list and/or the
core libraries committee. What's important here is to make sure that the
requester understands what the process is (either by explaining in the issue,
or by having a well documented policy). This allows the requester to understand
how best to push forward and improve his/her request or, alternatively, to drop
the matter.

__Have a backup maintainer that will take over if you are unavailable.__ If you
are generally busy allow him to release at least patch level changes without
directly consulting you. For example, [see Roman's Haskell
will](https://ro-che.info/articles/2014-02-08-my-haskell-will).

__Be polite.__ With very rare exceptions, requesters honestly want to make your
project better. Often times, that's obvious. In those cases, it's helpful to
provide a quick comment like "thanks for the feedback." The difficult one is
dealing with people who are not obviously trying to make the project better.
"Your stupid code broke my build, and you're a bad person" would be an example
of a difficult issue to deal with. Some suggestions:

* Never retaliate in kind; take the higher ground
* Don't allow their tone to take away from the points above. You should still acknowledge an issue, though you absolutely may decide to say "your bug report was not worded in a helpful way, doesn't provide meaningful information, and feels offensive. Please open a new issue with more details and a different tone and I'll be happy to help you."
* People sometimes have a bad day; try to extend them some courtesy
* Language barriers online are **massive**. Someone may not be a native speaker of the language you're using, and may imply something they didn't intend. We all know that text loses a lot of emotions, especially sarcasm. And I can admit that multiple times I've written on issues via my cell phone and put in typos that could have been read quite badly.
