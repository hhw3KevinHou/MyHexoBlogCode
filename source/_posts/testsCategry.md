---
title: Blue-green Deployments, A/B Testing, and Canary Releases
tags: 
- Blue-green Deployments  
- A/B Testing 
- Canary Releases
categories: test
---

## Blue Green Deployments

Pleaes see Martin Fowler’s link about blue-green deployments. It gives the overall gist. It’s basically a technique for releasing your application in a predictable manner with an goal of reducing any downtime associated with a release. It’s a quick way to prime your app before releasing, and also quickly roll back if you find issues.

Simply, you have two identical environments (infrastructure) with the “green” environment hosting the current production apps (app1 version1, app2 version1, app3 version1 for example):

![](greendeployment.png)

Now, when you’re ready to make a change to app2 for example and upgrade it to v2, you’d do so in the “blue environment”. In that environment you deploy the new version of the app, run smoke tests, and any other tests (including those to exercise/prime the OS, cache, CPU, etc). When things look good, you change the loadbalancer/reverse proxy/router to point to the blue environment:

![](bluedeployment.png)

You monitor for any failures or exceptions because of the release. If everything looks good, you can eventually shut down the green environment and use it to stage any new releases. If not, you can quickly rollback to the green environment by pointing the loadbalancer back.

Sounds good in theory. But there are things to watch out for.

Long running transactions in the green environment. When you switch over to blue, you have to gracefully handle those outstanding transactions as well as the new ones. This also can become troublesome if your DB backends cannot handle this (see below)
Enterprise deployments are not typically amenable to “microservice” style deployments – that is, you may have a hybrid of microservice style apps, and some traditional, difficult-to-change-apps working together. Coordinating between the two for a blue-green deployment can still lead to downtime
Database migrations can get really tricky and would have to be migrated/rolledback alongside the app deployments. There are good tools and techniques for doing this, but in an environment with traditional RDBMS, NoSQL, and file-system backed DBs, these things really need to be thought through ahead of time; blindly saying you’re doing Blue Green deployments doesn’t help anything – actually could hurt.
You need to have the infrastructure to do this
If you try to do this on non-isolated infrastructure (VMs, Docker, etc), you run the risk of destroying your blue AND green environments

As I’ve said, there are good techniques to overcome these challenges and make this deployment style work out very nicely, including plugging into a continuous deployment pipeline, but don’t jump into it trivially.
## A/B Testing

A/B testing is NOT blue-green deployments. I’ve run into groups that mistake this. A/B testing is a way of testing features in your application for various reasons like usability, popularity, noticeability, etc, and how those factors influence the bottom line. It’s usually associated with UI parts of the app, but of course the backend services need to be available to do this. You can implement this with application-level switches (ie, smart logic that knows when to display certain UI controls), static switches (in the application), and also using Canary releases (as discussed below).

![](abtesting.png)

The difference between blue-green deployments and A/B testing is A/B testing is for measuring functionality in the app. Blue-green deployments is about releasing new software safely and rolling back predictably. You can obviously combine them: use blue-green deployments to deploy new features in an app that can be used for A/B testing.

## Canary releases

Lastly, Canary releases are a way of sending out a new version of your app into production that plays the role of a “canary” to get an idea of how it will perform (integrate with other apps, CPU, memory, disk usage, etc). It’s another release strategy that can mitigate the fact that regardless of the immense level of testing you do in lower environments you will still have some bugs in production. Canary releases let you test the waters before pulling the trigger on a full release.

![](canarydeployment.png)

The faster feedback you get, the faster you can fail the deployment, or proceed cautiously. For some of the same reasons as the blue-green deployments, be careful of things above to watch out for; ie, database changes can still trip you up.
## Summary

All of these strategies can be implemented regardless of whether you’re using a particular cloud technology. But as you can imagine, technologies such as Docker and Kubernetes can be greatly helpful (if not even have provisions built in) for implementing these strategies. For example, OpenShift and Fabric8 greatly simplifies using Docker and Kubernetes by providing the tooling necessary to use these technologies without having to worry about the underlying details.


ref. http://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/