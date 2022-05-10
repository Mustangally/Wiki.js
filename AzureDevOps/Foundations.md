---
title: DevOps Foundations
description: 
published: true
date: 2022-05-10T15:59:56.989Z
tags: devops
editor: markdown
dateCreated: 2022-05-09T23:04:22.284Z
---

# What is DevOps

"DevOps is the union of people, process, and products to enable continuous delivery of value to our end users."

A key concept we'll address later but that we should take-away from this quote is the "continuous delivery". A major goal of DevOps is to do away with the long term release cycles (weeks or even months between iterations) and break these up into smaller bits that build on each other. This can often mean multiple releases a week or even per day!

## Time Before DevOps

Obviously the goal of a business is to deliver value to an end user. Within that business, you'll have developers who are trying to create new features and make changes to the product. On the flip side, you have the IT operations teams who are trying to minimize changes to protect availability and security.

This conflict in agendas often caused these long running projects that took months at a time to eventually deliver something tangible. This made change extremely slow and the company's product was not agile in any sense. This can cause more problems that we may not see coming, like customer needs or expectations changing over time. This can and has led to lost time and money as teams work on projects that become obsolete.

It was also very common for software products to be built in a monolithic architecture that did not allow for easy change on one part or another. You would see these cascading effects throughout the product for each change that was made.

## The Goal of DevOps

From a high level perspective, the DevOps philosophy seeks to solve this fundamental problem. We are trying to increase the agility or responsiveness of the development process, without compromising the stability and security of the product.

Of course achieving this goal is not simple and it has a new set of skills and tools. If you're going to have a middleman between Ops and Dev, you're going to need people who can "speak both languages". You're also going to need to adopt of develop new processes that address the agility and stability concerns with product updates.

You may have heard of the "Agile" development life cycle or process. Essentially this is an outline of the intended process in DevOps. It lays out a plan for how new releases are supposed to go, and the steps needed to do this properly. In DevOps we use the term Continuous Improvement/Continuous Delivery, or CI/CD. This is the fundamental philosophy that addresses the need for common and responsive updates and deliveries to the end user.

![agile_image.png](/agile_image.png)


## Architecture Change

We spoke about the old "monolithic" design of products, and the downsides of that. So what's the solution? Well to address the problem we can try to de-couple each component of the product into what we call "microservices".

Microservices are essentially smaller, somewhat isolated, building blocks of an application. We design and deploy each component seperately, and then logically connect them in the deployment process. What this allows us to do is modify and update each component seperately, without the concern of breaking the rest of the application. This makes bug fixes and feature updates far more seamless and changes can be made much faster.

An object oriented way of looking at this is comparing a VM to a container. While before we may have deployed an entire application to a given VM, we are now deploying each component of the application into its own container, and linking those containers together to provide the service as a whole.

### Practical Example

Lets look at this in practical terms because it may be easier to understand.

Lets say you have an application that has a web-based front end, that connects to a back end database for data storage and querying. Now lets say you see a spike in activity due to the holiday season or some other event, and now your database can't keep up with the requests coming in, but your web front-end is fine as all it's doing is waiting on a response from the database.

In the monolithic model, we'd need to redesign the entire application to expand the database, or modify the VM too allow for more available resources. This can be a lengthy process that may not be able to respond in time, and will inevitably result in lost profit as your application becomes slow and unresponsive.

In the newer "microservice" model, we would have deployed the DB and the web-app into seperate containers, that can be spun up and down independantly of eachother. This allows us to quickly spin up more database containers to address the incoming requests, and thus spreading the load seamlessly with no downtime, and in a matter of seconds.

## Waterfall

What is "waterfall"? Well this was the "standard way" that things used to be done. Think about it this way. There will always be a customer that is going to have a set of requirements. The company will then come up with a design to meet the needs laid out by the customer. Once we have the design created, we can then move on to developing, testing, and finally delivery. So you see how there are different distinct steps, that need to be completed in order. This is what we refer to as a "waterfall".

The issue with this process is it can be very slow, and it can take quite a long time. Depending on the timeframe, the customers' requirements may have changed in the meantime between design and delivery.

## Agile Process

To resolve the issues that come up from the "waterfall" methodology, we developed something called the "Agile" development process. The image above is a good visualization of the core idea. But what wasn't fully explained is that we use that process in far smaller increments. We would be doing that process on a smaller scale so that the process doesn't take long.

The focus here is the focus on the customer and interaction with them, as well as having working software that is responsive and quick to change when needed. We're also focusing on collaboration with the customer along the way so that they have constant input in the ongoing development of the product they're using.

# Scrum & Kanban

So in this whole process, how do we keep track of things? We need to have a utility to track the progress of projects and what we have going on. This is where things like Scrum and Kanban come in. 

## Kanban

Kanban you can look at somewhat like a step by step task management utility. You'll have stages in the process, and you'll have tasks for goals for the project. They'll all start on one end of the board, and throughout the process move across to the other end of your board. This is how we track the progress of development items and goals.

We look at this work plan, and it is a continuous process. There are not time windows or start and finish points. We have an "unending" stream of tasks that are making their way through the process. An important limiter here then is a "Work In Progress" limit. This is an agreed upon number of items that can be in process at a given time. No other item can start working through the process until there is a slot available for it. 

We still have a build, test, and deploy phase model, but we're controlling the number of items that can be in each stage at a given time, and thus the entire process between start and end. A key point to remember here is that we "pull" and item along, we don't "push" the item along. What this means is that if the deploy process is allotted a 3 item limit, when that phase has an empty spot, it will "pull" an item that is done with the test phase. This is in opposition to the test phase "pushing" the item into the deploy phase when it is finished testing.

So why do we have this WIP limit? Well that's how we keep the timeline to a shorter period. If we can limit the number of tasks in the pipeline, then we can make sure we're devoting resources to completing it, rather than it getting lost in a giant pot of tasks that all are competing for time. This allows us to bring our timeline down dramatically, sometimes to mere days or hours, rather than weeks or months.

## Scum

With the "scrum" mindset, the development team create what are called "sprints". These sprints are windows of time, something like 1 to 4 weeks, or anywhere in between. You'll then figure out how a quantitative representation of the workforce available. Let's say we refer to it as "points". The team has a 10 point work capacity for a 2 week sprint. The team will have a meeting at the beginning of the sprint where they'll look at the project backlog, and the points per task, and then assign the higher priority items to the sprint up to the points capacity allotted. Over the course of the sprint, developers will be pulling items into the next step (design to dev, dev to test, etc.) until they are done.

Don't think however that we can't combine the two concepts if we need. For example you can use the outline and structure of a Kanban Board to track the progress of a scrum sprint. These are just utilities, and we can use them as we see fit and however works best for our team.

# GIT & CI/CD

## GIT

The concept of GIT is a tool that allows us to control the "constant, incremental value" of a product, and the file tracking needed when multiple people are working on a project and constantly evolving. At its core, it helps us with version and source control of a given file.

Now when we say "GIT", you don't need to automatically think "GitHub". Yes GitHub uses the GIT technology, but it is not exclusive. You can use GIT in other spaces and with other offerings, and so is more of an overarching technology and tool.

So at the base, we have a "Git Repository". This is essentially a library of all of our code files for the project or team. Every developer is going to have their own local copy of the repo local to their machine (typically), and each of these copies will initially be identical, and reference a central source repo. We'll sync our local copy to our centralized repo to be sure we're up to date. 

We'll go into GIT in depth later, but it's important to understand it's necessity and use case. The last thing with this to understand for now is a "commit". Committing your code is essentially the act of synchronizing your local copy and the changes you've made to the central repo so that others can now see what you've done. You'll typically commit to a separate branch from the main, but we'll go into that at another time.

## CI/CD

Once you've committed your code, we can put it through something called a "CI Pipeline". What this is going to do is automate a series of tasks that help us verify the changes made, and address any problems. Our pipeline could be configured to do things like check security, build the code and test it, create an image for a container, and then upload the image to a registry. This whole process helps us automate checks and identify issues in an efficient manner, and significantly enables our ability to enact change quickly.

Now there's the next step, Continuous Delivery or "CD". This is the idea of automating the delivery and deployment of verified and functional code. So let's pick up at the end of the CI pipeline where we uploaded an image to a registry. Well now we'll enter the CD pipeline, and the act of uploading an image is a trigger to begin this process. The CD pipeline can automate things like building the needed environment/infrastructure, deploy the image or artifact to that environment, then test that application or deployment to make sure it's working. At this point it's common to have a "gate" where a someone needs to come along and check everything that has happened before it's approved and allowed to move on to the next stages before it is delivered to the customer. 
