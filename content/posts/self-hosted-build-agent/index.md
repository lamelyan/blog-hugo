+++
title = 'Self Hosted Build Agent for CI/CD pipeline'
date = 2024-06-21T01:24:43-05:00
draft = true 
featured_image = 'devops-azure-side-by-side.jpg'
toc = true
tags = ['ci-cd', 'azure', 'yaml']
+++

# Context

So you have a typical project with a backend API and a frontend app built with React or Angular. 
You want to ensure that every time you push changes to production, they are thoroughly tested and 
deployed. To achieve this, you set up a CI/CD pipeline that handles the building, testing, and 
deployment processes automatically.

At first, everything works smoothly, but as your project grows, you notice the pipeline is taking 
longer to complete. Eventually, it might even timeout. Why is this happening, and how can it be fixed?

## Nomenclature

Let's start out with nomenclature.

### What is a pipeline?

A CI/CD pipeline is a process that automates the steps of software development, from writing
code to deploying it to production, ensuring new code changes are automatically tested and 
integrated for reliable and faster updates. This pipeline is created by writing scripts or 
configuration files that define the steps for building, testing, and deploying code, which 
are then executed by CI/CD tools like Jenkins, GitHub Actions, or Azure Pipelines.

### What is a build agent?

A build agent in DevOps world is a dedicated machine (or a most likely a docker container)
that runs before mentioned scripts that build, test and deploy your application. 


## Why would you want to host your own build agent?

Build agents can become a bottleneck, as their performance directly affects the speed of the entire pipeline.

Microsoft provides you with their hosted agents that come with
[pre-configured](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#software) software.


There are [several reasons](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#capabilities-and-limitations)
to host your own agent, and in a project I worked on, the hardware specs
of hosted agents were insufficient for our workload.

At present moment Microsoft-hosted agent 
is provisioned with 2 core CPU, 7 GB of RAM, and 14 GB of SSD disk space,
see [more details here](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#hardware).


For small projects that don't require many resources this might be sufficient. 
However, for projects that involve more process-intensive tasks, the hosted agent configuration most
likely won't suffice.


An example of a process-intensive task is running tests for a front-end Angular application. This task,
in particular, can be memory-intensive. End-to-end tests that run with[Blog.yaml](..%2F..%2F..%2F..%2F..%2F.warp%2Flaunch_configurations%2FBlog.yaml) Cypress are another example. These tests are both CPU and memory-intensive.

On our project, the end-to-end tests were not able to complete in the alloted time.
The pipeline would reach the timeout of 60 minutes.  

This is why we chose to set up a self-hosted agent on a machine with enough memory to 
finish running the tests.[warp config.yaml](..%2F..%2F..%2F..%2F..%2F.warp%2Flaunch_configurations%2Fwarp%20config.yaml)


## Set up VMs that run your agent

As for setting up an agent, you can either spin up a single virtual machine or use a virtual machine 
scale set([VMSS](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/scale-set-agents?view=azure-devops)).
The latter will scale up with demand, making it a good choice if you have multiple CI/CD 
jobs that need to run simultaneously. Of course, you need to consider the cost of a scale set versus a single machine.

Once you have your VM or VMSS, you need to get the agent software running on the machine. 
Virtual machines do not have the agent running by default. However, Microsoft provides an 
extension for this purpose. Just enable the extension, and you will have a build agent running on your virtual machine.
![vmss-extension](vmss-extension.jpg)



## Connect VMs to Azure DevOps

Now that you have your virtual machines up and running, it's time to
configure Azure DevOps to use them in CI/CD pipeline. 

First step is to create a new "Agent Pool"
![agent-pool-create.png](agent-pool-create.png)

In the newly created agent pool, connect to the virtual machines that you created. 
![agent-pool-configure.png](agent-pool-configure.png)

## Use the new agent pool in your CI/CD builds

To use the newly created agent, set the agent pool, in your CI/CD configuration.
![pipeline-set-agent-pool.png](pipeline-set-agent-pool.png)

Now when you run the CI/CD pipeline, you should see agents being utilized.
![devops-azure-side-by-side.jpg](devops-azure-side-by-side.jpg)

