---
title: DevOps Foundations
description: 
published: true
date: 2022-05-09T23:04:22.284Z
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

You may have heard of the "Agile" development life cycle or process. Essentially this is an outline of the intended process in DevOps. It lays out a plan for how new releases are supposed to go, and the steps needed to do this properly.
![agile_image.png](/agile_image.png)

In DevOps we use the term Continuous Improvement/Continuous Delivery, or CI/CD. This is the fundamental philosophy that addresses the need for common and responsive updates and deliveries to the end user.

## Architecture Change

We spoke about the old "monolithic" design of products, and the downsides of that. So what's the solution? Well to address the problem we can try to de-couple each component of the product into what we call "microservices".

Microservices are essentially smaller, somewhat isolated, building blocks of an application. We design and deploy each component seperately, and then logically connect them in the deployment process. What this allows us to do is modify and update each component seperately, without the concern of breaking the rest of the application. This makes bug fixes and feature updates far more seamless and changes can be made much faster.

An object oriented way of looking at this is comparing a VM to a container. While before we may have deployed an entire application to a given VM, we are now deploying each component of the application into its own container, and linking those containers together to provide the service as a whole.

### Practical Example

Lets look at this in practical terms because it may be easier to understand.

Lets say you have an application that has a web-based front end, that connects to a back end database for data storage and querying. Now lets say you see a spike in activity due to the holiday season or some other event, and now your database can't keep up with the requests coming in, but your web front-end is fine as all it's doing is waiting on a response from the database.

In the monolithic model, we'd need to redesign the entire application to expand the database, or modify the VM too allow for more available resources. This can be a lengthy process that may not be able to respond in time, and will inevitably result in lost profit as your application becomes slow and unresponsive.

In the newer "microservice" model, we would have deployed the DB and the web-app into seperate containers, that can be spun up and down independantly of eachother. This allows us to quickly spin up more database containers to address the incoming requests, and thus spreading the load seamlessly with no downtime, and in a matter of seconds.