---
layout: post
title: How Everest orchestrates pseudo tab-switching
summary: Deep dive into the ideas and implementation of a memory optimization technique used in Everest.
cover: /images/posts/2018-09-19-everest-state-switching/header.png
---

_**Note**: This article was originally published on [the DEV Community](https://dev.to/rohit/how-everest-orchestrates-pseudo-tab-switching-49gp){:target="_blank"}_.

Everest is a REST API testing client that I've been working on this year. It's written in JavaFX and aims to be a **lighter, open-source** alternative to Electron-based options like _Postman_. It occupied the **#2 spot on GitHub's Java Trending** for a week back in May this year when I released the first alpha. Today, I'll be talking about a memory optimization technique that I've implemented in the most recent alpha release.

I'm calling it _pseudo tab-switching_. Why? Because I'm an engineer and we like to give fancy names to simple stuff.

This article is quite technical and some stuff is done in a certain way because JavaFX demands so. However, the concept itself is framework/lanuguage-agnostic and you can use it with any other tech stack and make better use of your resources.

A disclaimer before we begin: I'm a student. I'm still attending university and have no professional experience. I have literally conjured up these ideas out of thin air and implemented them because they sounded promising. If you are an experienced developer, you may find this rather simple and obvious. Nonetheless, this was easily the most complex thing I've implemented in Everest and I'm excited to share it with you. Any feedback is welcome!

# The Problem
![dashboard](/images/posts/2018-09-19-everest-state-switching/dashboard.jpg)

This is the Dashboard in Everest. You can compose and send requests and also view their responses within it. You can have multiple of them open in separate tabs.

Prior to pseudo tab-switching _(lets call it PTS, shall we?)_, for every new tab that you opened in Everest, a new Dashboard was created. That includes the view and the controller. Just a handful of tabs would push the memory usage north of 400MB.
This was really depressing and demotivating for me. At one point I even thought of removing the tabs altogether but that would mean giving in to a few electrons in my laptop and you won't be reading this article.

# The Eureka moment
About two months back, as I was dreaming on a separate thread of my brain during a university lecture, I had an idea:

> What if I load just _one_ Dashboard and change the content within it every time the user switches to a different tab?

Makes sense right? All tabs show the same attributes (API endpoint, HTTP method, status code and so on) just with different values.

This would give the illusion of switching between tabs but you'd actually just be switching between _states_ of the Dashboard. This would not only require way less memory but also facilitate better re-use of allocated memory. And that's where the name comes from!

Sounds simple, eh?
> "Why did you have to write an article about this, Rohit? My cat could do this while asleep.", you'd say.

![cat typing](https://media.giphy.com/media/o0vwzuFwCGAFO/giphy.gif)

Well, you have one smart cat. Jokes aside, implementing this logic was not a smooth ride and I ran into some design issues that I had not anticipated but ended up with some solutions that I'm quite happy with and proud of.

# The Challenges
So, barely able to contain my excitement, I bunked the remainder of the day's lectures, came home and fired up IntelliJ IDEA to implement my idea. (pun totally intended)

I wrote a new `DashboardState` class which would hold the values contained by all the UI elements ie the URL, the params, the HTTP method and so on. Everest already had a `HomeWindowController` class which handled the tabbing logic among other stuff. Before PTS, whenever a new tab was opened, HWC would simply create a new Dashboard and set it as the content for the new tab.

But since we've decided to use a single Dashboard and switch between its states, I wrote a new method in `DashboardController` called `setState(DashboardState state)`. Now, every time a new tab is opened, I create a new, blank DashboardState object and set that as the state of the Dashboard using this method. HWC also holds a `HashMap` to keep track of which state belongs to which tab.

I also wrote a nifty little `reset()` method in `DashboardController` which clears the Dashboard of the previous tab's state.

By now, we can properly display a state in the Dashboard. But when you switch from tab A to tab B and return to A again, how do we display A again? This is something I realized quite late. Before making the switch from A to B, we need to store the state of A in the aforementioned map.

So I wrote another getState() method in `DashboardController` which returns the current state of the Dashboard, which can then be saved. Simply put, when the user shifts back to tab A, HWC finds its state from the map and then uses `setState()` to apply it to the Dashboard thereby completing the switch.

So far so good. But a big monster lurked by as Rohit rejoiced in his pseudo tab-switching glory..

# The Monster
Before we get to the big problem, we need to understand how Everest handles HTTP requests. The requests obviously need to be made on a background thread. But Everest also needs to show a nice loading animation while the request is being made. To make this possible, I use JavaFX's excellent `Service` API. Not only does it allow me to make HTTP requests on a background thread, but also provides me with methods to update the UI concurrently. These include `setOnRunning()`, `setOnSucceeded()` and `setOnFailed()` which accept **lambdas** that update the UI.

Everest uses `setOnRunning()` to trigger the loading animation of the Dashboard, `setOnSucceeded()` to display the response (status code, body, elapsed time and so on) and `setOnFailed()`, the error messages. What I intend to establish here is that these lambdas act on the Dashboard.

But, if the user initiates a request in tab A and switches to tab B **while** the request is still running on the Dashboard, `setOnSucceeded()` would display tab A's result when tab B is selected.

The lambdas, somehow, need to be removed on a tab switch. But, we also need to keep the request running because if it is terminated or even halted, the utility of multi-tabbing ceases to exist.

Here's an elegant solution that I came up with, which I'm quite proud of.

# Beating the Monster
Remember that getState() method from `DashboardController`? I added a simple check within that to see what the Dashboard is currently doing. If a request is running, it will take that Service object, and hand it over to the `DashboardState`.

Now comes the clever bit: DS decommissions the lambdas that modified the Dashboard and adds its own which modify its own members!

For example, if `DashboardController`'s lambda for`setOnSucceeded()`displayed the status code of the response by modifying the text value of a Label onscreen, the corresponding one set by `DashboardState` would modify the `statusCode` attribute within itself.

![PTS in action](/images/posts/2018-09-19-everest-state-switching/pts-in-action.gif)

Now, when the user switches back to tab A, A's state will be applied to the Dashboard which in turn will have been updated by the lambdas set by DashboardState!

TLDR? PTS allows you to switch between tabs even when you've got requests running in one or more of them.

# The Gains 💪
![comparison](/images/posts/2018-09-19-everest-state-switching/comparison.jpg)

This comparison was made between Alpha 1.2 _(without PTS)_ and the upcoming Alpha 1.4 _(with PTS)_ with 7 tabs in each instance. With more tabs, the difference can become even larger since v1.4 has to only create another instance of a `DashboardState` rather than a full Dashboard like v1.2 does.

There is still scope for tiny improvements here and there, which will be done once we hit a feature-lock, which should happen by the end of this year or early next year.

# Summing it up
It all boils down to this simple algorithm:
- Save the state of the Dashboard
- Reset the Dashboard
- If a new tab is being opened, create and then apply a blank new DashboardState to the Dashboard
- If switching back to an existing tab, fetch its state from the map and apply it to the dashboard

Here's a formal-ish definition for pseudo tab-switching:
> If you have multiple UI elements that house a common UI element, then instead of creating multiple instances of the latter, create just one and share it between multiple instances of the former by switching between states.

# "Okay, now shut up Rohit!"
Yes, I hear you! This has been a very long post with a galaxy worth of jargon. I was going to make some announcements and ask a couple of questions but lets save it for next time.

Check out Everest on GitHub and try out Alpha 1.3. Let your developer friends know about it. Stay tuned for future updates as we're approaching a stable release early next year.

I hope you found something of value here. PTS can be useful even if you're doing front-end web development or native mobile app development.

Thank you for reading this! Leave a comment if you have suggestions, questions or anything else to say.

You may now Alt + Tab back to your text editor.
