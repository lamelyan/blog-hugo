+++
title = 'Database Change Management'
date = 2024-05-07T15:37:06-05:00
featured_image = 'azure-ci-cd-grate.png'
draft = true 
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



##  Solution



Use tools to automate this process.  Developers will only need to check in their scripts to a specified directory and the rest will be done by CI/CD pipeline.

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
