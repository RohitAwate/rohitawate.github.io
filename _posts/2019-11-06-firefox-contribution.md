---
layout: post
title: My First Contribution to Firefox
summary: An account of my first contribution to SpiderMonkey, Firefox's JavaScript engine.
categories: Open-Source Mozilla Firefox JavaScript
cover: /images/posts/2019-11-06-firefox-contribution/header.jpg
---

Since the past couple of years, I've open-sourced [most of my projects](https://github.com/RohitAwate){:target="_blank"} and also contributed to a few small ones. However, I've always wanted to contribute to a large, popular open-source project. I finally got around to doing that last month: I submitted a patch to Mozilla Firefox's JavaScript engine, SpiderMonkey, which was accepted on November 2.

I use Firefox every day. I appreciate and believe in the values and principles of privacy and an open Internet that Mozilla holds. Also, I have the highest amount of respect for people that volunteer in such projects. Thus, it felt great to contribute back!

This post serves two purposes:
- to document my experience with the hopes of inspiring people to contribute to Mozilla _(or other major open-source projects, for that matter)_
- to serve as a guide for someone making their first contribution to Mozilla since they have a pretty involved process and I don't want you to repeat the same mistakes as me!

# Finding a Bug
My patch fixes [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1589072){:target="_blank"}, which I found via [Codetribute](https://codetribute.mozilla.org/){:target="_blank"}. You can use the 'good first bug' filter to find beginner-friendly bugs. Codetribute only lists bugs; they actually reside on Mozilla's bug tracker, [BugZilla](https://bugzilla.mozilla.org/home){:target="_blank"}. Once you find a bug that you find interesting, check if someone else is already working on it. If not, add a comment that you wish to work on it.

You'll have to search for the instructions to get the source code, build the project, run tests and so on. For example, here's [SpiderMonkey's getting started guide](https://wiki.mozilla.org/JavaScript:New_to_SpiderMonkey){:target="_blank"}. This will vary depending on which project/module of Firefox you're contributing to.

# The Bug I Fixed

My contribution improves the errors reported by the JavaScript parser. Luckily, I got to work on a brand-new feature of JavaScript called numeric separators. This allows you to make your long numeric literals more readable by adding underscores between digits. This feature just shipped in Firefox 70 in late October 2019 and my patch will be live in Firefox 72.

```javascript
// Hard to read
let i = 1000000;

// Numeric separators improve readability
let i = 1_000_000;
```
_This feature is so new that Jekyll's Markdown parser, kramdown, is messing the syntax highlighting above!_

The [ES6 specification](https://github.com/tc39/proposal-numeric-separator){:target="_blank"} only allows a single underscore as a numeric separator between two digits. Additionally, a numeric literal must not end with an underscore. Thus, the following lines of code are illegal:

```javascript
let i = 100__0;

let j = 100_;
```

If you run this code under Firefox 70, you'll see the same error in both cases:

![ff70](/images/posts/2019-11-06-firefox-contribution/ff70.png)

Makes sense, right? However, the SpiderMonkey team wanted separate error messages for these cases. Following is a screenshot from Firefox Nightly, which includes my patch:

![ff72](/images/posts/2019-11-06-firefox-contribution/ff72.png)

These error messages are contextually aware and more in line with what the programmer would expect.

# Writing the Fix

Thankfully, Mozilla's [Jason Orendorff](https://twitter.com/jorendorff/){:target="_blank"} had provided detailed instructions on the BugZilla thread regarding the fix. Hence, it was just a matter of a few lines of C++. It was really simple.

You can view the patch [here](https://hg.mozilla.org/mozilla-central/rev/08d0bf739bad){:target="_blank"}.

If you have any doubts, just leave a comment on the BugZilla thread or reach out to the respective team on IRC. Mozilla's community is incredibly welcoming, helpful, smart and patient. Don't hesitate to ask questions. Communication is key. That's one of the most important things I learned in this process.

# Creating a Patch

Once you've made the changes, run the tests and are ready to submit, you can commit to the local Mercurial repository:
```c
// view the changed files
hg status

// view your changes
hg diff

// stage all of your changes
hg add .

// commit the staged changes
hg commit -m "Bug 1589072 - Improve numeric separators error messages"
```

For the commit message, use the above format. That number is the ID of the bug and the following message is its title, both from BugZilla.

# Submitting the Patch

This is the tough part and where I messed up the most. Mozilla uses it's own infrastructure and thus it's not as easy as opening a Pull Request on GitHub.

First, you need to submit your patch for review. Mozilla uses [Phabricator](https://phabricator.services.mozilla.com/){:target="_blank"} for this purpose. In order to submit your patch there, you need to use a command-line tool called `moz-phab`. Follow [this guide](https://moz-conduit.readthedocs.io/en/latest/phabricator-user.html#setting-up-mozphab){:target="_blank"} for setting up your Phabricator account and to install `moz-phab` locally.

Next, open a terminal and `cd` into the Firefox repository. Here, you can simply run `moz-phab` and it will push your changes to Phabricator and create a revision. The link to it will appear in your terminal.

For more information about using Phabricator, check out this [workflow walkthrough](https://moz-conduit.readthedocs.io/en/latest/walkthrough.html){:target="_blank"}.

# Code Review

Now, you need to wait for someone from the team to review your patch. They might request some changes or make some suggestions. Make the necessary changes. Again, if you have any doubts or questions, communicate with the team and get them cleared!

# Submitting Changes to your Patch

This is where you need to exercise caution. You might assume that you just need to run `hg commit` and `moz-phab` again to push your changes to Phabricator. If you do this, you'll end up creating a _completely new_ revision on Phabricator. This is the mistake I made.

You shouldn't create a new commit. Instead, add the changes to your original commit. You can do this using:
```c
hg commit --amend
```
Don't add the `-m` flag. Just run the above command, which will open your text editor asking you for a commit message. Add that; it can be the same as before. On the following lines, add this:

```c
Differential revision: <link-to-original-phabricator-revision>

// for example:
Differential revision: https://phabricator.services.mozilla.com/D51134
```

This tells `moz-phab` that this commit is a revision to your original patch. Consequently, it adds these changes to the same revision. You will be able to see your revision on Phabricator now.

# Acceptance and Landing

If the reviewer is satisfied with your changes, s/he will accept your patch. Next, you need to wait for someone to 'land' your patch. I'm not entirely sure but I think that means committing your patch to the [central repository](https://hg.mozilla.org/mozilla-central){:target="_blank"}. One of Mozilla's sheriffs will do that for you since you most likely don't have commit rights if you're reading this post.

Once that's done, well, congratulations! Not only did you work through your first patch and submit it, but you also got it accepted! You can check out your change live in the Firefox Nightly build.

![the-office-dancing-gif](https://media.giphy.com/media/AoVyddXBybVjq/giphy.gif)

# Conclusion

I hope this post served one of its goals _(or maybe both)_: either inspiring or helping you. I enjoyed this process and am proud of my patch, however little and simple it was. I hope to contribute regularly to Firefox henceforth.

We use open-source software every day, knowingly or not. If you have the time and skills, please contribute! The feeling of giving back is amazing.

That's it for today, see you in the next one!

_PS: I would like to thank my friend, [Jaydeep Borkar](https://jaydeepborkar.github.io/){:target="_blank"}, whose contribution to [spaCy](https://github.com/explosion/spaCy){:target="_blank"} inspired me to do this!_