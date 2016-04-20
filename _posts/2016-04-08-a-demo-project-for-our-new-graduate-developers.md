---
title: "Turning graduate developers into productive consultants"
author: nieldewet
date: 2016-04-20 17:00:00 +0200
---

In February this year, eight fresh-faced graduate developers joined our team.
To teach them the skills and technology they need to effectively serve our
clients, we decided to run a small, but substantial, internal project. The
intended outcome was to produce a valuable piece of software and give them a
chance to experience the challenges typical of a software development project
in an environment where failure was not fatal.<!--more-->

#### The mission

The objective of this project was to create a simple memory game to help new
team members learn the names of our roughly 180 colleagues. As we integrated
the system with our internal instance of [MyWorkLife](http://myworklife.com),
to access photos and basic information about our colleagues, it was necessary
for players to log in. Players were either presented with four photos to match
to a name, or four names to match to a photo. The game also included
instructions and a leader board.

Our team consisted of eight developers: three in our Cape Town office and five
in Johannesburg. We also had two graduate business analysts and a product
owner, with myself playing the role of team lead and Scrum master. Several
other colleagues provided support and coaching and we are grateful to everyone
who was willing to participate in user interviews and test the system.

#### The objectives

The main objectives of this exercise were to give the developers experience in
agile software development, working in team and using the common technologies
typically used by our Java consultants. Above all, we focused on extreme
programming practices like pair programming and test driven development (TDD).

Working in weekly sprints, the team had plenty of opportunity to practice
working in a Scrum team, with four sprints of one week each. Each week started
with planning and ended with a review and a retrospective. Every morning we
used video conference facilities to have a stand up and the team relied heavily
on Slack for communication between regions.

#### Rules

Emphasising the importance of continuous integration and delivery, the first
task we focused on set up a build pipeline in Jenkins to manage a one-click
release to our staging server using Docker images, which had to be ready to
deploy at all times.

To ensure we built the right product right, our business analysts applied UX
principles and conducted user interviews and tests. The feedback from this was
used to update the designs implemented by the developers.

We also needed insight into how users used the application. At first we used
logging, but later extended it by using Logstash, Elasticsearch and Kibana.

I used code reviews and pair programming to facilitate coaching, with pair and
[mob programming](https://en.wikipedia.org/wiki/Mob_programming) encouraged as
this was a very effective way for a relatively large team to collaborate on a
small code base, while learning to use many new tools and technologies.

To ensure exposure to project governance, we ran the project according to BSG's
governance processes and with the support of our Projects Office. This was a
very useful experience for the business analysts and also greatly contributed
to the successful delivery of the project.

#### What we learned

When we started the project everything was unfamiliar to the team and the only
thing we set up beforehand were two virtual machines and an empty repository on
GitHub. We could have created scaffolding with dependencies and basic
configuration, but we agreed that even though the progress on features was
slower, the team gained a deeper understanding of the technologies.

What the team struggled with the most was learning how to write good tests and
practise TDD. Most of the graduates had never written an automated test before
and we went through many cycles of demonstrating, pairing, reviewing and
coaching before slowly but surely the ice started to thaw and they started to
grasp testing. We kept it simple and focused only on unit testing and
handcrafted our own stubs to isolate code. It took a long time before the tests
reached a level of quality that added value and agility, but we did it!

With the team distributed across regions, another key experience was learning
to collaborate in a distributed team. We placed a lot of focus on our stand
ups, and reflected almost daily on how we could work together more effectively.
Different Slack channels were used for announcements and collaboration. Google
Hangouts, with screen sharing, was used to pair and mob across regions.

Other real world experiences gained included deploying a system with active
users, making sure the system was available for testing, doing database
migrations and integrating with other systems.

> “I have learnt the concepts of Java 8 and build tools such as Maven, Jenkins
> and Docker and the Spring and Hibernate frameworks. Along the way I have also
> learnt the benefits and organisation of a scrum team.” _-Marvin_

> “This project has taught me so much about being a software developer in an
> agile environment. Besides the new technologies I learnt, the most valuable
> concept I took home was how to use TDD and how it affects your code design.”
> _-Patrick_

> “Great experience practising Agile development, through facilitating
> stand-ups, constant engagement with Product Owner  to the importance of the
> SCRUM board moving efficiently. PO work and the importance of keeping Risks
> and Issues up to date and structured correctly.” _-Justin, Business Analyst_

> “The Noma project taught me how to refactor my code and apply TDD as well as
> how to code in collaboration with a big group of people.” _-Nicole_

> “From being on this project I learned that constant communication is very
> vital in team projects.” _-Ntokozo_

#### Would we do it again?

The project was a great success and the product delivered is fairly useful.
There are numerous improvements that can be made, but that is why we chose this
project in the first place - there was more than enough scope to keep the team
busy for the full duration of the exercise. The biggest success was that
everyone on the team gained valuable experience in working in a distributed
team, with some of today's most important technologies, and practised agile
principles and extreme programming.
