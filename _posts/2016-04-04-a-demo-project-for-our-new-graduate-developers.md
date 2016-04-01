---
title: "A demo project for our new graduate developers"
author: nieldewet
date: 2016-04-04 08:00:00 +0200
---

In February eight new graduate developers joined our team. To ensure readiness
for the different project environments that they would be going to we decided
to run a small but substantial internal project that would produce something
valuable and expose them to real world challenges in a safe environment where
failure is not fatal. <!--more-->

#### The project

For this project we decided to create a simple memory game to help new team
members learn the names of our roughly 180 colleagues. We integrated with our
internal instance of [http://myworklife.com](MyWorkLife) to get the photos and
basic information of our colleagues. In the game you are either presented with
four photos and name to match, or a photo and four names. The game also
includes instructions and a leaderboard. Because data from MyWorkLife is
exposed users are required to log in.

Our team consisted of eight developers; three at our Cape Town office, and five
at our Johannesburg office. We also had two graduate business analysts, a
product owner, and I played the role of team lead and Scrum master.

#### The outcomes

The main objectives of this exercise was to give the developers experience in
agile software development, working in team, and using the common technologies
typically used by our Java consultants. Above all we focussed on the extreme
programming practices like pair programming and test driven development. They
had plenty of opportunity to practice working in a Scrum team because we had
four sprints of one week each. We started each week with planning and ended it
with a review and a retrospective. Every morning we used a video conference to
have a stand up. The team relied heavily Slack for communication between
regions.

#### Principles

We emphasised the importance continuous integration and continuous delivery.
The first tasks were focussed on setting up a build pipeline in Jenkins that
is able to release to our staging server in one click using Docker images. The
system has to be ready to deploy at all times.

To ensure that we built the right product right our business analysts applied
UX principles and conducted many user interviews and tests. The feedback was
then used to update the designs and implemented by the developers.

Server Monitoring
Projects office.

#### The learning
Everything was unfamiliar at first.
Application integration.
Collaborating across regions.
#### Would we do it again?

