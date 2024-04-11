+++
title = 'Self Hosted Build Agent'
date = 2024-04-11T09:24:43-05:00
draft = true 
featured_image = 'conventional-commits.jpg'
toc = true
+++


# Why would you want to host your build agent?

Azure DevOps hosted agents might not have enough resources to run the tests.
The tests for the front-end Angular application in particular can be memory intensive.
On the project that I worked on, these end-to-end tests were not able to run on the default build agent.
The test would reach the agent timeout of 60 minutes and never complete.
We had to set up a self-hosted agent on a machine that has enough memory to finish running all the tests. 



