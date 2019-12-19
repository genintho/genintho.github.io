---
layout: post
title:  A Guide to Successfully Completing a Large Refactoring
date: 2018-01-22 19:00:00
---

## A Refactoring Tale

Very frequently, refactoring starts the same way. One engineer learns/thinks about something. A (naive) prototype gets built. It is demoed, presented to a few people. A pull request is created, updated and merged. Et voilà, all your problems will be solved by this miraculous new code. Fast forward a few weeks, the new/better way of doing things has been used marginally more. An attempt to use it in more tricky places fails because of unforeseen difficulty, fear of breaking things, or lack of time. The original author of the patch has moved on to some other kool new thing, is ashamed of it, or forgets about it. Fast forward a few weeks, (new) people exposed to the code are confused. Why is there 2 ways of doing the same thing? Which one should they use? What is the plan to phase out one of them? Trying something totally new is going to make things better!

![xkcd](https://imgs.xkcd.com/comics/standards.png)

Hipmunk used to be pretty accustomed with this situation, and my experience working at other places leads me to think it is a general way of approaching refactoring and introducing new libraries/paradigms in the code. But it does not have to be this way. With a little bit of organization, things can be much smoother.

## CommonJS
Hipmunk JavaScript code is organized into modules following CommonJS. These modules get bundled, optimized, polyfilled. We are using ES6, React/Redux, and Jest for unit tests. In short, we are using a very classic 2017 frontend stack and build chain.

But most of these tools did not exist when Hipmunk first started. At the time, using jQuery, global namespaces/variables and some kind of templating engine was the way to go.

2 years ago, CommonJS modules were introduced in our code base. It took this long, hundreds of commits, pull requests and deploys (and a few rollbacks 😫) to fully convert our code base. Along the way we learned/confirmed a few lessons that can be applied to any kind of large scale refactoring.

## Understand the Problem

Regardless of the type of refactoring, it is important to think deep and hard about it before going too far.
What *real* problem are you trying to solve? Is it worth solving? What is your time budget for that? etc

Thinking in terms of ROI is also important. Frequently, a problem is worth being solved only in principle, but in practice it is a complete waste of time. People are notoriously bad at time estimates. This famous xkcd board can be a good reference point. 

![xkcd](https://imgs.xkcd.com/comics/is_it_worth_the_time.png)

So shaving 5 min off a weekly task needs to be done in less than 21 hours (3 work days) for it to pay off over *5 years*! 
Keep this in mind before starting projects that will take months of work before being done.

Think in terms of distruption: will it increase complexity in the code base until completion? What needs to be done to mitigate this?

Come up with a list of pros and cons. Think hard about them, and go back to them after writing some prototypes.

In our case, we justified the CommonJS migration with the following:
- ability to write better unit tests
- ability to do some server side rendering using the same code as the client
- ability to use more 3rd party libraries and integrate them more efficiently
- ability to leverage better build tools that can help make our application better (Webpack split point, tree shaking, 
eslint, etc)

Cons:
- every file will need to be modified in some way
- dual build system during the transition period
- long and tedious refactoring
- potentially complex refactoring steps when some of our singletons are involved


## Get Buy-In

Every big refactoring/modernization project starts in the mind of a few engineers. But to be successful it needs to be embraced by the whole team, including non-technical stakeholders.

You want to get buy-in from the rest of the engineering team so they can help or at least not do things that will make the refactoring project more complex.

You want buy-in from non-technical people so they can understand why you are working on this project instead of directly on the product. Time can be carved out of the roadmap so people can focus on the project instead of trying to "jam it in".

People will have a higher tolerance to the eventual bugs, issues, or problems that will inevitably be created by the changes. 


## Have a Strategy

Often, large scale refactors get pushback because the size of the project makes them look impossible to do. That is the case when you look at them from the outside, without really thinking how it could be done in details.

Like most problems, if you divide them into smaller problems things get more manageable. And the key to the success of the refactoring is how you are going to cut the problem and in which order are you going to tackle the pieces.

Most of the time, the first piece is to have some sort of proof of concept of what your new code should be. The next step is frequently to build some kind of backward compatibility layer (more on that later). Then, you need to identify all the pieces that need to be changed and how they relate to each other.

Having a methodical, repeatable process to convert the code base really helped us make progress every week.

For our project, we were lucky that all our "jQuery era" modules were namespaced with some global identifier. After some bash and grep voodoo, we came up with a list of modules to convert. Tickets were created for each of them.

Then, we created a webpack entry point for our different page and started to convert the module needed for this file.
If a module could be totally converted, great we could close the ticket. If some other file was depending on it, we marked it in the ticket.

![Ticket Tree](/img/2018-01-22/ticket_tree.png)

By repeating this process, we were able to slowly convert our code and build a tree of the different modules that were left to be converted.


## Ensure Backward Compatibility

Any kind of large refactoring project is done incrementally. If you are building from scratch, then it is not a refactoring, but a rewrite :). The incremental approach requires that you think about backward compatibility. How is the new system going to co-exist with the old one? Is it possible? Do you need to write some code to make the transition smoother?

Depending on the project, working out the details of this can be the difference between a smooth ride or a path to hell.

For our CommonJS project we had to make sure our code was backward compatible. The first step was to find a way to use the code in the global namespace as we were, and with CommonJS.

So let's say a function generates a guid. We use to use it in our code by doing `hm.utils.guid`. In the file containing the code of the function, we simply added at the end:

```
module.exports = guid;
```

With this line, the guid function can now be used under CommonJS. But this code will throw a JavaScript error if used the old way. We fixed it by adding
```
window.module = {exports: null};
```

before any JavaScript is imported.


## Track Progress

Tracking progress of the project and communicating it (loudly) to the team helps make sure nobody forgets about this refactoring project but also keeps the team motivated. Important milestones should be celebrated.

For our CommonJS transformation, we kept progress by running some searches on our code base with some powerful regular expressions. While not being 100% accurate, it was a quick and dirty way to get a sense of where we were. By running the script every week, and sharing the results by emailing the team, it also gave a sense of momentum.

![email progress](/img/2018-01-22/email_progress.jpg)

We also used our bug tracker. We created a dedicated project and a release to keep track of progress and avoid people stepping on each other. We then created a ticket for each module that needed to be converted.

![Release progress](/img/2018-01-22/jira_release.png)


## Make It a Priority
Because this kind of project can take a long time, it is very easy for large refactoring project to lose steam, get side tracked, or put on hold and forgotten. Without an incredibly motivated team, things are going to go in between. From time to time, someone is going to say, "It would be great if someone would finish this. I have no time for it because I have this new feature to do". There will always be new features to do, and nobody will have time for it, except if you make this project a priority.

And if you have created or thought about a way to keep track of progress, it is easy to create a KPI.

Each week, month, release, sprint, you must make progress on the project. And because every byte counts, even the smallest progress is better than no progress. It keeps people motivated. It shows that this project really matters. It shows that you really care about your technical debt, not just talk about it.


## Be Ready for Some Pain
As Helmuth von Moltke would say, "No plan survives first contact with the enemy." So despite your best planning and analysis, you need to be ready for some pain.

The refactoring is going to take time to be done. Keeping working at it.
Bugs or major outages might happen. Have a rollback strategy in place to reduce their impact.
People are going to lose motivation or not see the point. Keep evangelizing.

It took us hundreds of commits and 2 years to finish our refactoring but we did it. And now, using the same methodology, we are starting our migration from Python 2.7 to 3.6!