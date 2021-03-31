---
layout: post
title:  "Reasoning about Agile from first principles"
date:   2021-03-31 21:00:00 +0800
categories: agile 
---

After being part of a Scrum team for more than a year, I realised that I have not thought deeply enough about Agile. 

Having attended Agile and ScrumMaster courses, my mind operated in the fashion of "Agile is this..., not this..." and "Scrum is this.., not this...", without understanding the whys. So, I decided to put some thought to understand and reason about Agile from first principles, hoping to have a better understanding of it.

This post was written specifically in the context of software development as I am not familiar with the application and usage of Agile in other fields. But I do hope that these insights are transferrable!

### The first principle
*Ultimately, software engineering is all about delivering value.* You could be part of a product team, working on a software product that satisfies the needs of your company or a client. You could even be part of a developer productivity team, working on tools to improve how your product team colleagues work. You are delivering value.

Naturally, the next question to ask ourselves is, what does it mean to deliver value? I think that it boils down to these three point:
1. Being able to correctly identify a problem to solve
1. Using the right solution to solve the problem
1. Getting better at solving problems

### Why the emphasis on solving problems?
All three points are essentially about solving problems. Why is that so? Fundamentally, how does being able to identify problems and solving them deliver value? 

By solving the right problems, you increase the amount of available resources either by freeing up existing resources or by increasing the total pool of resources. For example, you built a feature that allows your user to complete a task in a fraction of the time he/she usually takes. You just freed up his/her time (a resource).

### Correctly identifying the problem
Our understanding of a problem greatly affects what our solutions are. To avoid barking up the wrong tree, we need to correctly identify the problem before coming up with any solutions. **Imagine having spent many months working on a solution to find out that the problem you have previously identified was wrong or the context surrounding it has changed.** What a waste of resources.

### Using the right solution
Then, why is it important to use the right solution? 

1. Obviously it needs to be able to solve the problem. 
2. Because every solution has its associated costs. We have to ensure that its benefits outweigh its up-front and future costs.

**Imagine having spent many months working on a solution to find out that the solution you came up with does not actually solve the problem or it will be too expensive to maintain.** Again, a waste of resources.

### Getting better at solving the problems
Getting better at solving the right problems is delivering value in the second order. If you are this good at solving problems and delivering value now, imagine how it will be when you improve and get better at it.

### Beating up the waterfall model again
The waterfall model where we capture all our requirements into a specification document and implement them only after doing so, does not allow us to:
1. Correctly identify the problem to solve
1. Validate that our solution is right

You have to get them (the problem and the solution) right in one try, which is the time when you specified them in the document. Because of that, it is exceptionally bad at dealing with the **two bolded scenarios** that I have highlighted above. This creates a huge amount of risk because you would have wasted a huge amount of resource if such scenarios were to happen.    

The reality is that you will not get them right on your first try. Agile accepts this reality. 

### Agile, finally
I think that Agile enables us to
1. Be able to correctly identify a problem to solve
1. Use the right solution to solve the problem
1. Get better at solving problems

I could even say that these three points are the whys behind the [12 principles of the Agile manifesto](https://agilemanifesto.org/principles.html).

> Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.

This is the fundamental goal we want to achieve, delivering value.

> Welcome changing requirements, even late in development. Agile processes harness change for the customer's competitive advantage.

We may not correctly identify the problem in our first try, nor will the context surrounding the problem always stay the same. We should expect to course correct when necessary.

> Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.

By doing so, we accomplish a few things. We deliver value in the form of working software. We are also able to validate our solution, and as a result verify that we have correctly identified the problem. In the event that we were wrong, we course correct. The shorter this feedback loop is, the smaller amount of resources we would waste.

> Business people and developers must work together daily throughout the project.

Like the principle right above, we want to have a short feedback loop to know if we are wrong. 

> Build projects around motivated individuals. Give them the environment and support they need, and trust them to get the job done.

Teams with such composition are better able to accomplish the three points that I have been reiterating.

> The most efficient and effective method of conveying information to and within a development team is face-to-face conversation.

Face-to-face conversation has the shortest feedback loop out of any other forms of communication (e.g. email, messaging). This helps us to efficiently accomplish the three points.

> Working software is the primary measure of progress.

Progress is determined by problems correctly identified and rightly solved. This comes in the form of working software.

> Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.

The feedback loops are continuous, and the cycle of the loops should be consistent. 

> Continuous attention to technical excellence and good design enhances agility.

Whenever we have to course correct, technical excellence and good design allow changes to be made with lesser effort.

> Simplicity--the art of maximizing the amount of work not done--is essential.

We will have to course correct, and the amount of work not done = amount of work not having to spend the effort to change = amount of resources that can be spent on other areas.

> The best architectures, requirements, and designs emerge from self-organizing teams.

I think that self-organizing teams is another topic on its own. However, I think that individuals in a self-organizing team are more motivated. Like one of the earlier principles, motivated individuals are better able to accomplish those three points.

> At regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly.

By having reviewing ourselves, we create a feedback loop for the team to get better at solving problems.

### Feedback loops
We can see that a large part of Agile leverages on feedback loops. Two were mentioned:
1. The problem solving feedback loop
1. The feedback loop on the problem solving feedback loop

The problem solving feedback loop is where we 
1. Identify the problem 
1. Build our solution in the form of working software
1. Validate our solution, and we repeat

Whereas our second order feedback loop is where we
1. Look at our problem solving feedback loop
1. Determine if there are any improvements to be made
1. Implement these improvements, and we repeat

By combining these two feedback loops, we will only get better at solving problems. And I believe that this is what makes Agile so effective at delivering value.  