---
title: "The Beggining"
categories:
    - Experience
    - Web Development
tags:
    - nodejs
    - expressjs
    - mongodb
sidebar:
    nav : web_dev
---
Website Developement was a new world for me, the only "Websites" I had developed one page html and css websites. So, I knew the basics but I had no Idea how to start. It was decided that I would work on the Backend part of the website (Partially due to my bad design sense &#128517;)

![Backend Meme]({{ site.baseurl }}/assets/images/backend.png)

So I started with learning Nodejs and ExpressJs, Tutorials, Documentation, I read a lot and thankfully my teammate who was working with me knew the basics of Expressjs already. So we quickly split up the backend into controllers, routes and models, a standard split.

Now, for those who don't know - Controllers include the functions that are run, Routes are the Path through which these functions are accessed from when requests come in and models are the database models we define.  
Yeah, even though Mongodb is a schemaless DB, we use Schemas using Mongoose for easier Data Manipulation. Makes me question the choice of using Mongodb.

Anyways, I started Working on the Events Page requests, CRUD stuff (Create, Read, Update, Delete). 

![mongo]({{ site.baseurl }}/assets/images/mongodb.png)

Being new to Mongodb and Databases in general I had some pretty serious misconceptions about finding data in a Collection (A group of documents). I used to think that for searching for an item in the database, the query searches through all the documents of a Database, But Mongodb has this nice thing called [Indexes](https://docs.mongodb.com/manual/indexes/) which is a seperate data structure and contains specific fields, Mongodb Indexes on '_id' field by default and you can [add more fields](https://docs.mongodb.com/manual/core/index-single/) if you want. This allows one to do CRUD operations quickly.  
Do keep in mind though that creating too many Indexes is also a bad Idea, each document has a field in each Index, so for large collections a lot of excess space would be required for the Indexes.

![Execution Stats]({{ site.baseurl }}/assets/images/execution_plan.png)

You can optimize Mongodb queries further by checking out the query Plans of the queries you're planning to execute. Do this using the [.explain()](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) method in the Mongodb shell. 
The queryPlanner mode will tell you if it is using IXSCAN (Index Scan), COLLSCAN (Collection Scan) etc.  
Running this function in executionStats mode allows you to find out the number of documents scanned and compare performance.  
Read More about the query plan [here](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)


Up Ahead : [Setting Up Notifications]({{ site.baseurl }}{% post_url 2020-09-18-notification %})