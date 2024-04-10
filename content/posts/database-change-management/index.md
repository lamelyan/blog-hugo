+++
title = 'Database Change Management'
date = 2024-04-10T15:37:06-05:00
featured_image = 'overview-design.png'
draft = true 
+++

**Problem**

We have a database changes (schema or data changes) that need to roll out to Production. We could have a developer connect to database directly and execute those scripts. However, developers might not (...should not) have direct read/write connection to Production database.

It would be best to have a process that enables us to answer questions such as:

what is the version of the database running in a specific environment?

what changes database scripts are being deployed with the next release?

what is the reason for specific script?

who created the sql script?

when was the script run and by whom?

who approved the script to be run?

**Solution**

Use tools to automate this process.  Developers will only need to check in their scripts to a specified directory and the rest will be done by CI/CD pipeline.

**Overview**

Using a tool called 'grate', we can run scripts in a planned manner.

When build pipeline kicks off, it looks at folder that contains scripts and runs the scripts that haven't been run already.


