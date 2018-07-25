# Guide to open source maintenance

Maintaining an open source project is not trivial. It requires not
only technical skill, but also social and time management
skills. Finding the balance between helping others contribute versus
implementing changes yourself, or between being helpful and being a
slave to your project, is tricky. And worse yet: it’s non-obvious when
starting an open source project that these problems even exist! The
goal of this guide is to elucidate this topic, and clarify options.

This guide is obviously non-binding in any way: anyone is free to
maintain their project in whatever way they wish. But hopefully this
guide will make it easier to maintain a project. Finally, this guide
is itself an open source project following the guidelines below. In
particular:

**Maintenance level:** dedicated

**Accepting maintainers**: yes, request via PR

## Maintenance level

By making something open source, you are *not* giving any guarantees
to users of the software of any kind. However, some people may expect
certain levels of maintenance unless told otherwise. To avoid
confusion and frustration on both the user and maintainer sides, it’s
a good idea to define your maintenance level explicitly, in the
project’s README. Here are some suggested "standard" levels you can
reference:

* **Abandonware**: this code is available as-is, but the author has no
  interest in it. If you find a use for it, great. But don’t expect
  PRs to be merged, or issues to be responded to.

* **Interested**: the author has some interest in the project, and may
  merge PRs/respond to issues. However, don’t expect much.

* **Happy to merge**: the author will review PRs and merge good ones,
  but is less likely to respond to issues

* **Dedicated**: the author will strive to review PRs promptly, and
  get back to issues in a reasonable timeframe, usually within a
  week. Of course, real life has its own demands, and vacations
  happen, so the timeframe may be delayed.

* **Highly supported**: there is a dedicated team of maintainers
  working to keep this project running smoothly. PRs will be reviewed
  and merged very quickly, and issues should be responded to within 48
  hours, barring unforeseen circumstances.

In addition to these levels, it’s also good to specify whether people
can request that they be added as maintainers as well. This could look
like:

* **No**: not interested in having other maintainers on the project

* **Yes, request via PR**: if you send a few good PRs, and say you’re
  interested in being a maintainer, you’ll get commit access

* **Yes, request via issue or email**: specify a contact method, and
  what the requirements are to get maintainer status

Again, these are just suggestions, and you can of course define your
own level (or leave this alone entirely!).

## Use standard contribution tools

You should make it as easy as possible to contribute to your
project. And this means you should use *standard approaches* whenever
possible, for two reasons:

* People have already learnt these approaches. Even if your new
  approach is better, do not underestimate the resistance people will
  have to learning something new! This isn’t stubbornness on anyone’s
  part, it’s a natural defense mechanism against needing to learn
  thousands of variations on the same theme.

* Standard approaches have been battle-tested to solve common problems
  people run into.

Here are some things you should consider:

* Put useful information into a README.md file in the root of your
  repo. In addition to information on how to use your project, it
  should give some pointers on how to contribute, even just as a link
  to another document. Make sure to include:

    * A list of project goals, to hopefully clarify whether a certain
      feature fits in with the project or not.

    * The maintenance level, mentioned above.

    * Any information about how you like to receive PRs. This can be
      highly opinionated, and provided as a link. Here’s my highly
      opinionated guide:
      [https://www.snoyman.com/blog/2017/06/how-to-send-me-a-pull-request](https://www.snoyman.com/blog/2017/06/how-to-send-me-a-pull-request)

* If you’re using Github or similar, provide issue and PR templates to
  help people file useful issues and complete PRs. As one point of
  reference, see Stack’s .github directory:
  [https://github.com/commercialhaskell/stack/tree/master/.github](https://github.com/commercialhaskell/stack/tree/master/.github)

* If you care about consistent styling, be sure to include a reference
  to the style guide.

* Add CI scripts that enforce what you care about. It’s much easier
  for a contributor to fight with Travis or AppVeyor than to have a
  higher-latency back-and-forth with the maintainer. CI can be used
  for styling, linting, and much more!

## Responding to issues

This ties in directly to the maintenance level. What I’ll describe
here is the ideal goal to strive for. But keep in mind what we
discussed earlier: you’re not a slave to your open source projects,
and have the right to say "I’m not going to deal with this now, or
ever." But *please* consider being upfront about such a decision!

* Triage quickly. If new issues come in, have a standard triage
  process, which will include:

    * Labeling the issue

    * Asking for more information (ideally, have a standard issue
      template that requests the most common info already)

    * Telling the issue reporter that it’s being looked into

* Define an owner, who is responsible for driving an issue
  forward. This is *vitally* important on multi-person projects. It’s
  too easy for issues to remain in limbo because no one knows who’s
  ultimately responsible for making a decision.

    * On Stack and Stackage, we’ve found a great approach is to have a
      team of people take turns being "on call" for initial triage,
      and to pass off issues to others when necessary.

* Define clear steps to closing the issue. It can be frustrating for
  everyone involved if there is no way to move forward. Get a
  reproducing test case, say "if this passes, we’ll close the issue,"
  etc.

* If issue filer unresponsive, close it. There is limited time in
  everyone’s life, and limited energy in the universe. Unnecessary
  open issues clog up people’s mental capacity. If you don’t hear
  after X amount of time, close with a comment "closing due to
  unresponsiveness." Ideally, explain this in the README or
  contributor guide; it’s not a personal slight, it’s just official
  policy.

## Say no when appropriate

You will receive feature requests and pull requests which don’t fit
the project’s goals. You’ll receive poorly written PRs. It helps no
one involved to drag out the process if you know this will never be
worked on. Say no upfront, ideally explaining why. If there’s a
possibility to fix the request/PR, leave it open. If it’s inherently
mismatched, close it out. (But please *don’t* mute the conversation,
that’s rude.)

Related to this: encourage contributors to open up issues before
submitting large PRs. A large PR adding a feature that won’t be
accepted is frustrating for everyone involved: contributors wasted
their time, and maintainers hate turning away work. Better to have a
design phase. (I mention this in my opinionated PR guide.)

## Labels issues for newcomers

Be sure to label issues that you think a newcomer will be able to work
on. It provides a gradual onramp to understanding the code base, and
getting PRs merged is a dopamine hit for everyone involved. Resist the
urge to knock out the easy issues yourself! You **will** spend more
time helping a new person with an easy issue than doing it yourself,
but it’s an investment in their future, as well as your project’s.

Opinionated recommendation: be free with the commit bit once people
prove themselves, they rarely abuse it, and you can always revert
something bad. I typically give out a commit bit (if I remember to)
after 2-3 successful PRs. (I think this was originally advocated by
Edward Kmett.)

### Encourage contribution

Whether explicitly in your README, or implicitly with how you
communicate, you should try to motivate people towards wanting to
contribute. There is little glory in day-to-day maintenance of an open
source project. You should give people getting involved as much in the
way of positive reinforcement as you can. Ideas include:

* They will be able to hone technical skills

* They will get access to mentorship from someone with more experience
  (see below)

* They will be empowered to fix problems that are hurting their work,
  instead of being blocked on an upstream maintainer

* It can work as resume building. Note that, unfortunately, some
  employers are *not* incredibly interested in FLOSS experience. But
  many others do weigh such activities heavily.

### Offer mentorship when possible

When you have the time and inclination to do it: make it clear that
you’re available to mentor people through issues. Make yourself
available on something like IRC or Gitter, as people prefer live chat
to the (perhaps more intimidating) communicate-via-Github-issue
route. Make it clear: you won’t be facing the wolves alone!

## Increase the bus factor

The above leads directly into the bus factor. For those unaware, the
bus factor is how many people need to be hit by a bus before your
project dies. In addition to spreading knowledge and enthusiasm about
your project, please be sure that:

* At least one other person has commit access to your repo if you’re
  gone, and upload access to wherever your project is deployed
  (e.g. Hackage).

* Set up a fallback plan if you aren’t available, either short term or
  long term. Consider having an
  [open source will](https://ro-che.info/articles/2014-02-08-my-haskell-will)

* Ideally make sure others know the code base well enough to maintain

## Difficult users

This is the most depressing part of the guide, which is why I’ve left
it till the end. There will ultimately be people who interact on an
open source project in less-than-ideal ways. Figuring out how to
respond to them is difficult to say the least. By identifying the
behavior, and having standard responses, I’m hoping to be able to
depersonalize it a bit. Furthermore, perhaps by writing this down,
those interested in participating in open source will learn some
behaviors *not* to demonstrate themselves.

### Poor bug reports

This one is all too common. Writing a perfect bug report is
*hard*. You need to figure out all of the context you need to provide,
without providing a wall of text. If you err on either side (too much
or too little), you’ll waste somebody’s time. Figuring out the balance
is hard. If you’re new to reporting issues, explain as such, and be
open to getting asked to rephrase the bug report and significant
constructive criticism.

Keep in mind: you’re asking for help from someone, your goal should
make it as easy as possible to provide that help.

### XY problem

I won’t waste time on redefining the XY problem,
[Wikipedia defines it well](https://en.wikipedia.org/wiki/XY_problem),
plus a good thread on
[Stack Exchange](https://meta.stackexchange.com/a/66378/291252). The
XY problem is usually a direct result of trying to be respectful of
people’s time, and not bore them with "unnecessary"
details. Unfortunately, in the XY problem, it often backfires.

There’s no sure-fire way to completely avoid it. If you’re trying to
figure out how to concatenate two strings, you don’t need to give a
backstory on the business goals of the app you’re writing. By
contrast, if you’re trying to read a file into a textual value, and
have figured out how to read it as a binary value, don’t ask "how do I
convert a binary value into text?" Most likely, the broader scope will
reveal better answers.

### Help vampires

I believe the term was originally crafted by Amy Hoy in the blog post
[Help Vampires: A Spotter’s Guide](http://slash7.com/2006/12/22/vampires/). Again,
I’ll pass on redefining the term here, as that blog post does a great
job of it. There’s a difficult line to walk when dealing with help
vampires. On the one hand, you want to foster a community where people
are able to ask questions and help each other. On the other hand, a
non-stop onslaught of tedious questions with no end in sight can crush
the spirits of contributors.

I can’t say that I have a perfect answer to dealing with help
vampires. Education about the problem is great, but by their very
nature, the help vampires are least likely to see the educating
material. Ultimately, it seems like there are two ways to deal with
the problem:

* Ignore it and let it fester. Contributors may end up offering help
  to the vampire, get sucked dry, and lose momentum on the
  project. Alternatively, no one may answer the help vampire, giving
  the project the (unfair) appearance of being unhelpful to struggling
  users.

* Address the problem head-on, and explain that the person in question
  needs to take more responsibility for solving their own
  problems. This is harder to pull off, but ultimately is worth
  it. Here’s a sample wording paraphrased from a real-life example:

This doesn't sound like a bug or soluble issue for the project
maintainers. Instead, this sounds like a request for consulting or
assistance. You might try an IRC channel or a Slack community to see
if someone can help you.

I mean this with no malice: Please keep the issues you file on GitHub
limited to actionable issues for the people working on the
project. We're here to improve the project. We cannot provide help
with open-ended project work. We are here to work on stuff that helps
all users of the project. If I fix a bug, that helps me and helps
everyone else using the project. If I help you with a specific,
non-project-related issue, that doesn't help any other users users. I
understand you probably didn't know this as a lot of these norms in
open source are unspoken and unwritten, so I'm trying to lay out some
of it so that you understand better going forward.

### Entitled complainers

Some users feel that they are entitled to some kind of solution for
their problem. Part of the goal for maintenance level is to set these
expectations correctly. As a semi-concrete example, imagine that
operating system XYZ seems to behave differently than many other
OSes. You don’t use XYZ, and you’re not particularly interested in
herculean efforts to make your project work for it. You may receive a
stream of complaints requesting/insisting/demanding that you look into
these issues.

Set expectations. State in the README that there are known issues with
XYZ. Explain that it requires more effort to support that
configuration than you have. And if you’re not inherently opposed to
XYZ support, mention that having a contributor on the team take
responsibility for XYZ support will help significantly with getting it
better maintained.

