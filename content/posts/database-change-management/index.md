+++
title = 'Database Change Management'
date = 2024-05-07T15:37:06-05:00
featured_image = 'azure-ci-cd-grate.png'
draft = false     
toc = true  
+++

So, you have database changes—either schema or data changes—that need to be implemented and rolled out across various environments.

One approach could be to connect directly to each database server and execute scripts. However, you might not—and indeed, probably should not—have direct read/write access to certain environments like beta or production.

Additionally, consider if a client posed the following questions; would you be able to easily respond?

- What is the state of the database in our Production environment?
- What database changes are being deployed with the next release?
- What is the reason for a specific change?
- When was a specific table or data changed?
- Who approved database update made three months ago?
 
It would be beneficial to have a process that systematically answers all these questions 
and relieves you of the mundane tasks, freeing you up for more engaging activities.


## Overview of the process

1. Version control your SQL scripts
2. Set up CI/CD pipeline to take script from version control and execute against a database in specified environment

### Version control SQL scripts
Why put SQL scripts into version control?

Committing SQL scripts to source control answers questions such as who created the change,
when it was made, and why. Of course, the level of detail depends on the information included 
in your commit messages. Besides the message itself, referencing the user story or bug number 
associated with the change can be beneficial. This practice further helps clarify the reason for the change.

Once this is done, anyone can look at version control history and figure out why this change was there.
This helpful not just for SQL scripts but for any code that effects application or infrastructure. 

### CI/CD Pipeline


### Tools

Let's lean on some tools to automate the process.
Ideally, developers will only need to worry about writing their SQL scripts to make changes to the database.
The deployment process should then automatically handle pushing these changes to the appropriate environments at designated times, with approvals from the appropriate parties.



If you are using Microsoft SQL Server, then Microsoft has a tool for you  [SQL Server Data Tools](https://visualstudio.microsoft.com/vs/features/ssdt/) 
to automate this process. However, this tool is Microsoft specific. 

Is there a more generic version of this that is tech stack agnostic?

One such option is the [Grate - SQL scripts migration runner](https://erikbra.github.io/grate/). 

Since this tool just executes SQL scripts, you can use it to deploy to variety of database. 

I was able to use Grate on a couple of projects that included PostgreSQL and MySQL databases.


### Data Migration workflow

After the SQL scripts are committed and pushed, we manually trigger a build of the pipeline called DataMigrations.

#### CI/CD pipeline process

Azure pipeline does the following:

Pull source from specified branch (that includes new SQL scripts)

Build an docker image. This image has everything we need to run data migration. It includes the grate tool along with the scripts that need to run.

Runs the scripts against database in specified environment

## Grate

Using a tool called grate, we can run scripts in a controlled manner. Here is a post that describes how grate processes scripts.

In the "deploy" step of the Data Migration pipeline, the grate tool looks at folder that contains scripts and runs the scripts that haven't been run already.



Once it runs the scripts, it also updates the version of the database.

### Notes about the Grate tool

How does crate tool know which version the database is on, which scripts it ran,  and their status?  The grate tool relies on it's own database tables. During the first run, the grate tool creates it's own schema with tables that helps with tracking of the database state. If you look in any instance of the Freshbyte database, there is a schema called 'grate'.



The three tables enable the tool to figure out the state of the database. It also allows us to troubleshoot. If we want we can query these tables to find out which scripts ran and which failed.
