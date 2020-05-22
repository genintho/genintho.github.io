---
layout: post
title: An attempt at interviewing for mentorship skills
date: 2020-05-15 19:00:00
---


One of the most popular criticsm for interview in tech is the lack of focus on soft skills.  At one my previous job, I had the opportunity to design a question test the mentorship skills of candidates.

We were looking for a tech lead for an infrastructure team. One of the duty of this person would be to mentor other team members, as well helping design new system. How can we test this during a 50min interview?

We thought that asking the people to review a design document was a good way to assess mentorship. As a senior member, reviewing the work of other is a big part of your duties. Even if it is not done intentionally, the feedback made during these reviews shape the culture of the team. What you notice, what you question and how you do it defines your leadership.

To introduce the question to candidates, we were saying the following:

> At $COMPANY_NAME$ we ask people to write a small design doc before starting to work on new projects. This document gets reviewed by the different leads and manager before the work can starts. We see it as a great way to mentor each others.. In this case, a junior engineer has been tasked to work on a new project that might require a new piece of infrastructure. He wrote a design document and is asking you to review it. What would you say? Please keep in mind this is a junior developer asking for feedback from a senior member of our team.

We provided the candidates with a laptop connected to a private Github repository with a "real" pull request containing a mardown document, waiting to be reviewed. The content of the document was 100% made-up and designed in a way to allow find plenty of potential comments: about the process followed, the decision made, the timeline, risks, etc.


##  Expectations

Internally, we talked a lot about what we wanted the canddiate to do during that session:
- ask questions about the business context. Some context is provided, but it is on purpose too undefined
- ask questions about what is already existing. This document propose to introduce a new technology while something already used in the company could be used and be as good
- notice the lack of care for security or privacy
- think about the monitoring, not just from an infrastructure perspective
- discuss the proposed rollout strategy
- discuss about the actual solution
- offer to help the "engineer" to figure out specific points
- bonus point if they actually write anything down on the pull request

## What did candidates do?

Overall is was pretty bad. We interviewed a dozen candidates and only one passed.

### skim through the document instead of actually reading it
It is easy to see the problem this creates when the whole interview is build upon the text. We left the candidate a few minutes to read the document in silence. Or prompt them to read it again when it was obvious they did not in the first place.

### went deep in the weeds very quickly
The document had the word Docker. Some candidate went bananas because of that. That's all they could think of. Nothing else matters we needed to go full in on Docker with every singe new tools you can think of. What the project did, what the company use had or even the scale had no importance.

### say nothing
Some candidate got stumped. They could not figure out the question. it was too diferent from what they were used to. Despite our attempts to make them confortable.

### did not seem to have ever done a review
This one was the truly suprising one. But it seems that despite being senior SRE or DevOps people, they were truly unfamiliar with the idea of reviewing something or discussing an implementation plan.


## What did we learn?
Regular tech interview obviously requires some skills from the interviewer. But testing only soft skills require a order of magnitude more. We constantly second guesses the questions, revises our dummy pull request, and strategy to explain the question or how to help the candidates. It requires a lot of empthaty and clarity on what the question is about. It also require some people that actually care about hiring. 
The originality of our question really destabilized candidates. They could not use a memorized series of tricks, like when answering a system-design or algorithm question. Critical thinking and communication became crucial skills, like they are on the job.






