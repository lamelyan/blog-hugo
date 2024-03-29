+++
title = 'Document vs Relational Database'
date = 2024-03-29T05:03:33-06:00
+++

# Overview

On my latest project, I had the opportunity to explore an alternative to relational databases: Document Databases. Document Databases belong to a category called NoSQL databases.

I found a couple of advantages of using a document database compared to a traditional relational database, which I will demonstrate with an examples below.



--- 

## Schema flexibility


Let's say you have a table that stores user's information.  Let's store just first and last name. What do we do when we design using relational database? We create a table users with two columns that store first and last name respectively.



### Relational way

![[Pasted image 20240328103445.png]]

At a later point, we decide that we also want to store user's twitter handle. Where would it go?  We could create a new column in user table or have a new table that stores user social media information. In either case, we need to update database schema.  PAIN! Deal with scripts and red tape that goes along with updating database schema in different environments.

![[Pasted image 20240328103500.png]]

### Document way

Let's see how it's done using document database. Using document database, we store data as a document. Specifically we store it as json object. This allows us for flexibility to change our schema without creating additional tables or columns. Just add a new property to the json object.

![[Pasted image 20240328103524.png]]


## Eliminate Data Transformation


One other goal of document database is to reduce data transformation. Let me demonstrate with example below.


### Relational way

Let's say we want to store a person's profile akin to LinkedIn.


![[Pasted image 20240328103620.png]]



### Shredding the data

What is typically done in a relational database is the normalization of data. We break down different characteristics of the information into their own tables with their own columns. Essentially, we 'shred' the data.

![[Pasted image 20240328103711.png]]

Now, to retrieve the information we need, we query data from multiple tables and hope for performant joins that don't cause delays.

Essentially, we pay the price twice: once by creating multiple tables with their own columns and relationships, and then with the need for queries from multiple tables. Both of these steps are prone to errors and performance issues.


### Document database

One of the features of a document database is to store the data the way you need it. If you're going to retrieve a user's profile information this way most of the time, then store it accordingly—as an entire profile stored as a JSON object.

![[Pasted image 20240328103727.png]]

Store it in one table and query it by user's id or email. Done!

## Conclusion

In this post, I highlighted just a couple of advantages that I see in opting for document databases. However, there are some disadvantages that I found. For example, while you can quickly select columns from a table and scan it for the data you need in a traditional relational database, since the JSON data is stored in one column in a document database, you need to view that JSON and write a query to parse it. Fortunately, with ChatGPT, writing these queries isn't a big deal.

There are other considerations...but that's a topic for next post. 






