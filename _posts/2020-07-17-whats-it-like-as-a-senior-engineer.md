---
layout: post
title: "What's it like as a Senior Engineer?"
excerpt: 'Ever built a tool no one used? I have. It sucked. Most of your time goes into identifying what needs to be built and how to build it. You have to research what the problem looks like. You talk to others and get everyone to agree on what needs to be done.'

tags:
- interviews
tweets: []

---

When I started working at Microsoft, fresh out of college, I loved coding. Writing code felt like the easiest way to build any cool thing that my brain could imagine.  When I thought about what I’d want to do for the rest of my life I thought that I just wanted to keep coding. During the next 11 years I became a Senior Engineer at Microsoft and moved on to work at Google and later Stripe. 

At these higher levels I still get to build, but I use a very different set of tools to do it. There’s a huge mindset shift needed when you go from junior to senior. Writing code becomes a minor part of the job. 

Ever built a tool no one used? I have. It sucked. At the senior levels most of your time goes into identifying **what** needs to be built and **how** to build it. You have to research what the problem looks like. You talk to others and get everyone to agree on what needs to be done. 

These are your new tools:
* Research the problem
* Design the solution
* Build consensus

# Research like a Detective
Fresh out of college you get handed tasks where the right answer is pretty straightforward. There isn’t much disagreement on what to do other than the occasional feedback in code reviews.

As you get more experienced your problems become more ambiguous. The path looks hazy. There are multiple routes you could take, but each one hides its own dragons. It’s not about coding anymore. Most of your work goes into research, and you can’t google the answer.

Research can take many forms. It usually involves a combination of reading code, reading documentation, and talking to people. Yes, actual human beings. In fact, that’s where most of the information you’ll need is locked away. Did you ever see Sherlock Holmes using search engines?  
![Sherlock](https://lh3.googleusercontent.com/8zyhAq2HSKAYAzysnJfbR_m6ZCBloCL5jbUmlwLEnJnmErsiHzd6gOEPtMpJZIp961AlNxUrdloY4B3cFFpVca5iH7Xb3wJgj0rFG7TLO60IGey4TaOJGZITimTXGv7U9hXqh-Pp)

There often is no single person who knows the answer you need. Five different people might hold five different pieces of the puzzle you’re assembling. And you don’t know who those five people are. And they don’t know which pieces you need.

You have to find them. Find them and ask the right questions to sift through their brains, uncovering the nuggets you need.

At Google Cloud Platform our customers would often contact our Technical Solutions Engineers for help when their environments broke. Those TSEs dug into the problems and fixed them. My manager had an idea: “Wouldn’t it be great if we could use AI to automate that process?” We had no clue how to do it. Didn’t even know if it was possible. Heck, we weren’t even sure what kind of problems customers were asking for help with. But that was the challenge my manager offered.

I accepted.

Now any AI solution for this kind of a problem requires lots of data. The AI needs to see many broken environments to understand what they look like. And as I searched around I realized we didn’t have that data, it was all locked away in the brains of those TSEs. You can’t train AI with that. 

I had to find the patterns. Maybe chatting with the TSEs would reveal something...

Me: “So, what type of problem do you usually face?”
TSE: “Eh, it’s something different every time”

Darn it, The AI future was looking bleak. 

Me: “Well, what do you do to solve it?” 
TSE: “It depends. Based on the problem, we’ll query one database or another. Then that’ll point us somewhere else, and we keep digging until we find what’s wrong. Then we fix it.”

No solid data on what problems they solve. No repeatable way to fix them. I was ready to give up.

Wait a second.

“Tell me more about these queries you run?”

What if I changed the problem? Maybe I didn’t have to fix those customer issues right off the bat. What if I helped TSEs debug the problems faster? I could automatically run the hundreds of potential queries the they tend to run and suggest “Hey, this one had a suspicious result. Maybe dig a bit deeper there?” That’s a lot of debugging the TSEs could avoid.

I could even extend this to collect the data needed for an actual an AI system. This had potential! The TSEs were excited. My team was excited. My manager was excited. We began coding.

# Design: The Art of Balance
With ambiguous problems there is no single right answer anymore. There might not be any answer. What you have is a pain point. It could be your customers’s pains, your team’s pains, or even your own pain. The existence of that pain is the problem. Your job is to remove that pain without introducing even greater pains.

There’s a funny thing about ambiguous problems: they don’t have a clear right answer. Every solution offers certain benefits and has certain downsides. The more of those you discover, the better you’ll be at balancing the tradeoffs you have to make. Some common trade offs to consider:

* How long will it take to develop the solution?
* What’s the opportunity cost?
* How risky is it? What happens if that thing fails?
* How much work will it be to maintain this going forwards?
* How far will it scale? How far does it need to?

With these ambiguous problems, sometimes the best answer can be “keep doing the thing we’ve been doing.” That was a tough lesson to learn. 

When I was a wee lad four years out of college, I had been asked to come up with a way to make our database upgrades less risky. The team would manually review all the planned changes to make sure they were safe, but once in a while a bug would slip though and the sound of pagers going off would fill the room as everyone frantically tried to fix it.

“Can we build a tool to catch those risky changes?” my manager asked me. Woah, this was a super open ended problem. Sweet! I was determined to not let him down. This required digging deep into database upgrade best practices (I even read a whole book on it cover to cover). I spent the winter holidays toiling away developing a prototype that could do upgrades safely. And it worked! Kinda.

When I showed my creation to my manager he was worried: “You know what, let’s just stick with doing things the way we do right now.” 

Ouch. 

It was a tough lesson on risk management, but he made the right call. A bug in my tool could have brought our entire service down. It wasn’t worth the risk.

![face palm](https://lh6.googleusercontent.com/-SpPwF1sWOPZD0HnYx0IqqluusAVritix1g7_NToe3s93jEmkPTsYhCTHJdPh7axMdLIN6gZ0fJ1_01AiPDputH2wPnMBXTDmgfxYIFAicsQeyAlq8Y9fRTxBFljmx9FZYi1Pdcy)

There were multiple lessons I learned that day:
* Consider how much risk any new project might add to the system
* It’s okay to fail. If you never fail then you’re not stretching yourself
* Get feedback early! 

To get that feedback communication is crucial. Tell people what you’re going to build before you build it and let them warn you about any pitfalls before you step into one. If I had shared that design with my manager before building it we would have cancelled the project weeks earlier. And I would have had a relaxing winter break.

But collecting feedback requires a soft skill: empathy. Can you understand why people disagree with you? What are they valuing differently?

You may not always agree with the feedback, but you have to understand it. Only then can you move forward with a new vision that everyone can get behind.

# Build Consensus
Getting that feedback and agreeing on the plan grows more important as your projects get bigger.

You may start off just having to get your manager to agree (he’s the one who gave you that ambiguous task). But you’ll need to build consensus with the rest of your team and even people outside your team who have a stake in your work. 

This requires communication skills, both to understand and be understood.

Once I was tasked with creating the next generation of our internal database management system. This was something many teams depended on, and our current solution would stop scaling a year or two down the line. My team had seven different people with eight different opinions about what the system should look like. That included my manager and skip level. Oy vey. 

First step was talking to them all to really understand their concerns and priorities. But there was another voice I wanted to hear from: our customers! This was meant to be a system for other engineering teams, how could I build a solution for them without understanding their problems?  It took a bit of digging to even figure out who those users were. This required another soft skill: The art of finding the person you need to talk to.

Eventually I got into a room with them. There they dropped the bombshell “We can’t really justify the work to migrate to any new system. The current one works well enough for us right now and we have more urgent problems to fix.” I talked to three different teams and got the same answer each time. Damn, what’s the point of building a solution if no one will use it?

A migration had to happen, soon the current system would stop meeting our reliability standards. There were a couple routes forwards:

* Politics: Get my management chain to convince their management chain to force the teams to migrate. Yuck
* Persuasion: Teach those teams why this pain that they won’t feel for a few years is more important to fix than the pains they’re facing today. That’s a hard thing to prove, and we’d have to make this case to many, many teams. That doesn’t scale well

There was a third option: change the constraints. What if I said ‘no’ to some of the features I’d been asked to add? Removing that let me design the system in a way that we could migrate all our customers automatically. There would be zero work required from them to migrate. We’d swap the engine with the car still zooming down the highway.

This was much more palatable. And by highlighting our user’s push back I convinced the other stakeholders to change drop those constraints as well.

![consensus](https://lh5.googleusercontent.com/uQ8pUq_dwFP6BBDO1d66HKZNGs5DrxtbLCoKc0NxnrsvpHOlh_C_Pi9yUcoU7XLi9TiYkRKy_4QItN3s3pWBVlNCfm2MZYyECTrdEHaJsQVTVxqmV4GPVzEFXJaRxpS5wqloltjj)

That’s the general flow of any project you work on at the senior levels: You research the problem, gather the pieces, understand the world better. You design a solution, collect feedback and adjust course as needed. Then the implementation begins.

So how do you learn all these skills? Experience. Jump out of the nest and flap your wings. If an opportunity shows up, take it. You won’t feel ready, no one does, but that’s what makes it a learning experience.

Ask for help. Listen to the answers you get. Keep trying. At the end of the project ask for feedback and use it to improve faster. You only learn these skills by practicing.

It’s a new world at the senior levels. You’ll still be building. But with new tools you’ll build bigger and better than before.

