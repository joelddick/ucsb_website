---
layout: default
permalink: /project/
title: Course Project
---

# Course Project

## Objective

The goal of the project is to gain hands-on experience in building and
deploying a scalable web service on the Internet. Using the latest web
technologies while learning how to tackle the scalability and fault-tolerance
concerns. This is a "learn by doing" course: the course project will form the
primary focus of the course with the lectures and discussion of research papers
providing background material. Each project will be conducted in an agile team
where students will build their own scalable, redundant web site using
fundamental web technologies and the Ruby on Rails framework.


## Getting Started

* Complete chapter 1 in the
  [Ruby on Rails Tutorial](https://www.railstutorial.org/book/beginning)
* Read the list of [project ideas](/project_ideas/).
* Add your own project suggestions.

## Project Sprint Schedule

All sprints end and begin with each week's lab session.

#### Sprint -1: September 22, 2016 -- September 28, 2016
* Install [Rails](http://rubyonrails.org/).
* Learn [Ruby](https://www.ruby-lang.org/en/).
* Complete [Ruby Code Academy](https://www.codecademy.com/tracks/ruby).
* Complete chapter 1 in the
  [Ruby on Rails Tutorial](https://www.railstutorial.org/book/beginning).
* Begin chapter 2 in the
  [Ruby on Rails Tutorial](https://www.railstutorial.org/book/toy_app).
* Form your team.

#### Sprint 0: September 28, 2016 -- October 5, 2016
* Complete Chapters 2 through 6 in the Ruby on Rails Tutorial.
* Determine your team's project, get approved by instructor.
* Learn [git](http://rogerdudler.github.io/git-guide/).

#### Sprint 1: October 5, 2016 -- October 12, 2016
* Complete chapters 7 through 10 in the Ruby on Rails Tutorial.
* Create and push the base rails application to your team's github repository.
* Learn TDD: get [Travis CI](http://docs.travis-ci.com) working with your
  github repository.
* Start writing stories for your project in pivotal tracker.
* Decide on a sprint commitment.
* Learn pair programming through pairing up on the first few stories.

#### Sprint 2: October 12, 2016 -- October 19, 2016
* Complete chapters 13 and 14 in the Ruby on Rails Tutorial.
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint.
* Learn EC2 and Amazon Web Console by deploying your initial application on
  Amazon EC2.

#### Sprint 3: October 19, 2016 -- October 26, 2016
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint.

#### Sprint 4: October 26, 2016 -- November 2, 2016
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint.
* Deploy a tsung instance on EC2.
* Write an initial tsung xml file to load test a simple action on your
  application.
* Define the "critical path" through your application (the set of pages that a
  common user will go through).
* Write a tsung xml file to exercise this critical path.
* Load test your application with tsung running on a m3.medium instance.
* Produce an initial set of graphs with your application's performance.

#### Sprint 5: November 2, 2016 -- November 9, 2016
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint. Your application should be mostly
  feature complete at this time and subsequent work should focus on polish.
* Test (and document) the effects of vertical scaling on your application with
  the critical path xml file.
* Deploy and test your application on a variety of load-balanced
  configurations, with the medium--large dataset.
* Test using Tsung, measure, and document.

#### Sprint 6: November 9, 2016 -- November 16, 2016
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint. Only polish is appropriate at this
  point.
* Create a medium--large dataset (~10,000+ records) using database seeds.
* Test (and document) the effects of horizontal scaling on your application
  with the critical path xml file.

#### Sprint 7: November 16, 2016 -- November 23, 2016
* Conduct a retrospective on how the last sprint went and how you can improve.
* Decide on a sprint commitment.
* Implement stories from the current sprint. Only polish is appropriate at this
  point.
* By the end of this sprint your project should be feature complete.
* Create a large dataset (more than 100,000 records).
* Inspect and optimize the way your application is interacting with the
  database. If necessary, write custom sql to optimize.
* Continue testing using Tsung, measure, and document.
* How many users can you handle?

#### Sprint 8: November 23, 2016 -- November 30, 2016 (No lab on the 23rd)
* Conduct a retrospective on how the last sprint went and how you can improve.
* If appropriate, create as large of a dataset as possible. How big can it be?
* Implement server-side caching (if you haven't already)
* Deploy your caching optimizations on a simple load-balanced configuration,
  with the large dataset, both with and without memcache.
*  Continue testing using Tsung, measure, and document.
* How many users can you handle?

#### Sprint 9: November 30, 2016 -- December 5, 2016 (Monday)
* Conduct a retrospective on how the last sprint went and how you can improve.
* Complete the project write-up (due Thursday, December 1).
* Prepare final presentation (present Monday, December 5).
