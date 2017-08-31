---
title: 'Program Like a Grown Up'
date: '20-05-2015 00:00'
taxonomy:
    tag:
        - 'your mom'
        - growth
        - maturity
---

I've been in this industry for about 10 years now. It's like surfing the Australian current in Finding Nemo. You can't get out for even a year or you feel left behind. Technology changes too fast and it's hard to keep up. By the time you really grasp something, a new thing comes out to replace it.

__Everyone was in this circle at some point:__

> "Brooooooo have you used Guard?"

> "Yeah Guard is great, but have you heard of Grunt?"

> "Grunt is awesome, but have yall heard of Gulp?"

They all look about the same, do the same thing and no one knows the real difference or benefit from one to the other except for us nerds. It's the [business card scene](https://www.youtube.com/watch?v=aZVkW9p-cCU) from *American Psycho*.

The rapid growth of our industry has created programmers trying so hard to keep up that they don't have time to grow and mature as business people who think for themselves. 

##Boundaries and growth

The younger children are, the more boundaries they need to succeed. They don't understand why eating loose change isn't a good idea, they just need a slap on the hand or to be told not to do it. The more mature, the less boundaries are necessary. Eventually they understand why the boundaries exist and what they are protecting them from, and at this point the boundaries are no longer needed. Where understanding abounds, boundaries vanish and creativity flourishes. 
 
I think the same is true for anything new, including the life of the programmers career. When they are a newborn programmer, they will succeed in a more controlled environment. During this season it's totally okay to watch a tutorial on the latest cool thing out there and jump in and apply it to their project, even when it doesn't quite fit. The more mature and experienced they become the more they can accurately assess the technical needs of their projects and apply the appropriate approach. 
  
## Grown ass children

So what happens is some programmers never get off of the teat. They become more knowledgable but still hold on to their boundaries. Knowledge != Wisdom. They acquire more knowledge but apply it within the context of those same limitations. They correct other programmers on the internet they don't even know by saying "You should be using X or Y" without having a clue what the needs of the application are. They can't see the big picture past their own technical dogma. They end up being really really good at the thing that provides no value to a business or to the end user. They do everything right, just because. They will work on something for 6 hours so that it's done "right" according to that guy in the tutorial when it could of been accomplished with the same result in 2 hours. Their massive egos result in twitter drama because they are constantly trying to one up each and they have zero empathy. They get all butt-hurt when their ideas are challenged and respond with aggression and anger. They never listen to other people and they seldom ask themselves "why?"

##Learn the thing, understand the why, apply accordingly

I enjoy constantly learning new technologies and technical principles/patterns/architectures (whatever you want to call them). In the past year I have done a lot of research on Domain Driven Design (DDD), Event Sourcing, CQRS and Microservices to name a few. It would be impractical to just rebuild the applications I maintain every time I learn a new thing. The reason why any of these principles exist in the first place is because they originated as common answers to common problems. If you understand the "why" you can apply where necessary or even where possible in a way that practically makes sense. 

####Patterns work for you, you don't work for them.

Below are a couple examples of how I practically and fairly quickly got to solving problems as I learned each thing:

#####Domain Driven Design 
Working for a large company with a very complex domain allows me to immediately apply principles like "ubiquitous language". I can view other departments I serve as the "domain experts" and I work with them more closely in order to completely understand the domain and create a model that is flexible enough to scale with company growth. 

#####Event Sourcing
I built an app using some of the principals of event sourcing but it's not at all a representation of this kind of architecture. I built it using Laravel and the architecture relies heavily on commands and events. There is a huge focus on having a truly read-only model that has no functionality that allows for deleting or updating records in the database. This is to prove the accuracy of all of our public website approvals from our regulatory department since our products are heavily regulated by the FDA. This borrows the idea of "immutable data" from event sourcing. Every record is logged and required to remain forever and remain unchanged. "Well actually, immutability is only achievable in a language that..." yeah yeah whatever man, no one has access to modify these records and that's what is important to the business here. 
 
#####CQRS 
 (Command Query Responsibility Segregation) is a pattern that uses a different model to update information than the model to read information. This is normally implemented in an event sourced application but again, I don't really care. I refactored an API to use Doctrine's data mapper for all of my reads which includes very complicated relationship nesting. For all of my writes I use Elequent models. Separating these made both models more efficient, resulted in less code, was much faster and the API is much much easier to maintain.
 
#####Microservices 
 This architecture means your application is composed of small, independent processes communicating with each other using language-agnostic APIs. Some of the benefits are that you can deploy these services independently, they are used over HTTP and can live on multiple servers. For a major project at work I created separate API's but that are all within a single Laravel app on the same server. Technically not microservices but they are independent de-coupled components with different endpoints that know nothing about each other. Each API has a very specific responsibility, so they are pretty "micro" and they are "services" soooooooo... I dunno. These are consumed by our company intranet, which is built on ExpressionEngine. These microservices include employee data from Active Directory through an LDAP connection, sending eblasts with Mandrill using Active Directory query based distribution groups, an ExpressionEngine API, and a service that searches within documents using DOS commands and Windows file system indexing. Doing it this way provides a ton of benefits and allows our IT department to use them for their .NET apps. But some guy who has read all of Martin Fowler's Microservices articles would be the first to say "well, actually..." 
 
 ##Take Away
 My whole point is, I have to make sure I stay focused on the primary end goal and I can switch back and forth between viewing the big picture and diving deep into the technical details. Being successful and separating yourself apart from the forum commenting twitter correcting purists isn't that hard. Just constantly ask yourself "why", what problem is this solving? How does this provide value? And surround yourself with people you trust who challenge you for your own good. Ignore the people still drinking breastmilk. 
 
 #####Sources:
* Event Sourcing and CQRS: [Greg Young](https://www.youtube.com/watch?v=JHGkaShoyNs) is a fantastic teacher so listen to anything by him
* Microservices: Ton of resources out there but read [this](http://martinfowler.com/articles/microservices.html) first by Martin Fowler
* Domain Driven Design: [Wiki](http://en.wikipedia.org/wiki/Domain-driven_design). You kind of just have to read [this book](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by Eric Evans!
   
 
 











