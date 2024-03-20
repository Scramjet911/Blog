---
title: "Setting Up Notifications"
categories:
    - Experience
    - Web Development
tags:
    - nodejs
    - expressjs
    - mongodb
    - agenda
excerpt: I started working on the notifications system on the backend
sidebar:
    nav : web_dev
---

![Notif meme]({{ site.baseurl }}/assets/images/getting_notified.png)

So, after the Events Controller was done I started working on the notifications system on the backend.  
Registered Users who save events on the website have an option to add notifications/reminders for the Events, so we need a system on the backend which checks the pending notifications and sends notifications to the users.  

This calls for a task scheduler module for nodejs, I looked into [cron](https://github.com/node-cron/node-cron#readme), [agenda](https://github.com/agenda/agenda#readme), [bull](https://github.com/OptimalBits/bull#readme) and some others and found that agenda would be the best fit for the job. It has persistence (The tasks are saved even on server restart), uses Mongodb as its database, is lightweight and is easy to use.

Thus, I started with Agenda and created the functions for adding new notification jobs requested by user. It was when I had to do the updation operations that I came across a problem...  
Agenda doesn't have a function to Edit a Job. You have to delete the existing job and add a new one, this wouldn't be a problem if I had the job id of the Job to delete. But I only had the User Id and had to find the job using it, which leads to a collection scan on the agenda collection (Inefficiency++) every time a notification is updated...  

I couldn't allow this to happen (Don't ask me why..) so I decided that I had to create an Index on the User Id on the agenda collection.
Now, we only had the Mongoose module installed on our server and after hours of Googling I found out how to modify a Mongodb created database using Mongoose's Mongodb driver Methods, instead of installing Mongodb.  
The way you do it is like this -
```javascript
    // Create an Empty Schema for the Agenda Collection
    let agendaSchema = new mongoose.Schema({},{collection:'agendaJobs', strict:false});
    // Create a Mongoose model for that collection
    let agendaModel = mongoose.model('agendaJobs',agendaSchema);
    // Add the Indexes if they don't Exist, the data of the jobs is stored in a data field by Agenda
    let indexes = await agendaModel.collection.getIndexes();
    if(!('user_Id' in indexes)){
        agendaModel.collection.createIndex({"data.userId":1},{name:'user_Id',sparse:'true'});
    }
```
Voila, You got yourself indexes...  

Still Reading? : [Pfft, Chat should be simple]({{ site.baseurl }}{% post_url 2020-09-19-chat %})