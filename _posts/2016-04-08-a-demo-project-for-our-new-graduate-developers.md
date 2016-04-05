---
title: "A demo project for our new graduate developers"
author: nieldewet
date: 2016-04-08 08:00:00 +0200
---

In February eight new graduate developers joined our team. To ensure readiness
for the different project environments that they would be going to we decided
to run a small but substantial internal project that would produce something
valuable and expose them to real world challenges in a safe environment where
failure is not fatal. <!--more-->

#### The project

For this project we decided to create a simple memory game to help new team
members learn the names of our roughly 180 colleagues. We integrated with our
internal instance of [MyWorkLife](http://myworklife.com) to get the photos and
basic information of our colleagues. In the game you are either presented with
four photos and a name to match, or a photo and four names. The game also
includes instructions and a leader board. Because data from MyWorkLife is
exposed users are required to log in.

Our team consisted of eight developers; three at our Cape Town office, and five
at our Johannesburg office. We also had two graduate business analysts, a
product owner, and I played the role of team lead and Scrum master.

#### The outcomes

The main objectives of this exercise was to give the developers experience in
agile software development, working in team, and using the common technologies
typically used by our Java consultants. Above all we focused on the extreme
programming practices like pair programming and test driven development (TDD). They
also had plenty of opportunity to practice working in a Scrum team because we had
four sprints of one week each. We started each week with planning and ended it
with a review and a retrospective. Every morning we used a video conference to
have a stand up. The team relied heavily Slack for communication between
regions.

#### Principles

We emphasised the importance continuous integration and continuous delivery.
The first tasks were focused on setting up a build pipeline in Jenkins that
is able to release to our staging server in one click using Docker images. The
system has to be ready to deploy at all times.

To ensure that we built the right product right our business analysts applied
UX principles and conducted many user interviews and tests. The feedback was
then used to update the designs and implemented by the developers.

We also needed insight into how our users are using the application. At first
we simply used logging, and then extended it by using Logstash, Elasticsearch
and Kibana.

I used code reviews and pair programming as tools to facilitate coaching. Pair
and [mob programming](https://en.wikipedia.org/wiki/Mob_programming) was
encouraged. This turned out to be a very effective way for a relatively large
team to collaborate on a very small code base while learning to use many new
tools and technologies because it avoid stepping on each other.

To give exposure to project governance we ran this project according to all the
BSG governance processes and with the support of our Projects Office. This was
a very useful experience for the business analysts and also greatly contributed
to the successful delivery of the project.

#### The learning

When we started the project everything was unfamiliar to the team. We didn't
set up anything beforehand, except two VM's and an empty repository on GitHub.
We could have created some scaffolding with dependencies and basic
configuration, but in hindsight we agree that even though the progress on
features was slower the team gained a deeper understanding of the various
technologies in this way.

Certainly the hardest thing to learn was how to write good tests, and
especially how to practice TDD. Most of the graduates had never written an
automated test before. We went through many cycles of demonstrating, pairing,
reviewing and coaching, and slowly but surely the ice started to thaw as they
started to grasp testing. We kept it simple and focused only on unit testing
and handcrafted our own stubs to isolate code. It took a long time before the
tests reached a level of quality that added value and agility, but we did it!

With the team distributed across regions another key experience was learning to
collaborate in a distributed team. We placed a lot of focus on our standups,
and reflected almost daily on how to work together more effectively. Different
Slack channels were used for announcements and collaboration, and Google
Hangouts with screen sharing was used to pair and mob across regions.

Some of the other real world experiences gained were deploying a system with
active users, making sure the system is available for testing, doing database
migrations, and integrating with another system.

> “I have learnt the concepts of Java 8, Build tools: Maven, Jenkins and Docker
> and the Spring and Hibernate frameworks, I have also along the way learnt the
> benefits and organisation of a scrum team.” _-Marvin_

> “This project has taught me so much about being a software developer in an
> agile environment. Besides the new technologies I learnt, the most valuable
> concept I take home was how to use TDD and how it affects your code design.”
> _-Patrick_

> “Great experience practising Agile development, through facilitating
> stand-ups, constant engagement with Product Owner  to the importance of the
> SCRUM board moving efficiently. PO work and the importance of keeping Risks &
> Issues up to date and structured correctly.” _-Justin, Business Analyst_

> “The Noma project taught me how to refactor my code and apply TDD as well as
> how to code in collaboration with a big group of people.” _-Nicole_

> “I learned from being on this project, is that constant communication is very
> vital in team projects.” _-Ntokozo_

#### Would we do it again?

The project was a great success and the product delivered is fairly useful.
There are definitely many further improvements that can be made, but that is
why we chose this project in the first place, because there is more than enough
scope to keep the team busy for the full duration of the exercise. The biggest
success was that everyone on the team gained valuable experience in working in
a distributed team, with some of today's most important technologies, and
practised agile principles and extreme programming.
