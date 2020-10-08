---
title: "Musings on Building a Data Catalog(ue)"
date: 2020-10-08T17:15:10+01:00
tags: ["programming", "bigdata", "engineering"]
draft: false
type: "post"
banner: "posts/2020-10-08/images/header.jpg"
---

In my previous role as Team Lead for the Big Data Team, at the top of my remit was to deliver a Data Catalog. In ernest we started working on bringing together the entire company's data estate under a catalog. Searchable, accessible, and updateable in a democratic manner.

A self-service two way marketplace for data within the company... Engineers would be able to access datasets programmatically.
Data Scientists would have 'cooked' versions of the datasets in self service analytics platforms like Databricks, and Jupyter notebooks. Dataset owners will be able to publish to the data lake and catalog with turnkey solutions, where they only needed to set up once and wash their hands off.

The company has all it's infrastructure in the AWS 'cloud' and as such used a lot of S3 for semi structured datasets like json files, log files, xml files, and even gzip files that contained simple txt files in some loose structure. How do you consistently present that to technical and non-technical users in interfaces, that provide consistent UX for everyone including other data engineers?

One thing people tend to overlook when talking about the role of a Data Engineer is the large component that is infrastructure engineering. As a data engineer, the role is not just to build tooling for data analysis, but also find efficiencies in the use of your infrastructure to achieve that goal. It took a lot of months to come up with a Atlas (yes that's what I called it), as I envisioned it as a map through which you could arrive at the data one needs.

Now let's consider the following environment (This is going to very AWS centric, where I can I will shed more light on other equivalent technologies). Those familiar with AWS would know what accounts are and what Virtual Private Clouds (VPCs) are. As the Big Data team we had to build tools to traverse accounts and VPCs.

## The Personas

This catalogue was going to be built to initially satisfy 3 broad use cases.

## Data Producers

From a producers perspective, they wanted to be able to continue with their BAU (churning out datasets from their day to day operations) with the added benefit of those datasets ending up in the catalog to be consumed by other teams. The data could be logs from a website that contains e.g click data, preferences, habits, cookie data whatever a website does to track usage from entry to exit (and sometimes beyond).

## Data Scientists

From a data scientists perspective, it would be nice to be able to access "cooked" or "cleaned" datasets from the mass of diverse data both useful and otherwise that was the data lake. It would be nice to access these datasets as they became available in whichever manner they chose. Either through using Databricks, Jupyter, a Commandline Tool, or even an IDE that connected directly to a Spark cluster.

## Data Engineers

Building a spark job should be as simple as querying an endpoint for the location of the data, plugging it into your job and transforming away. All at run time, because the catalogue should be able to tell you where it is and what the access methods, strategies, and guarantees should be. At some point it would also be nice to get the data as a stream too.

As we had already dipped our toes in the data lake concept, this would be a starting point. It became about understanding the boundaries of the data lake (if there were any). To understanding the infrastructure that held the lake together. (S3 buckets, mysql databases, oracle databases, ReST APIs, Redis Clusters, everything). This was the challenge to chart this infrastructure and how data arrived and churned within those places.

 I don't know how many parts this series is going to be but I would like to share my thoughts on how we approached this problem highlighting that I imagine this is stil work in progress but while I was still leading this project I feel like we made some really good progress to delivering a scalable solution.
