---
title: Healofy Architecture: Behind the scenes of 4 million daily active users
published: true
description: Healofy Architecture: Behind the scenes of 4 million daily active users
tags: healofy, architecture
//cover_image: https://direct_url_to_image.jpg

---



![](https://res.cloudinary.com/practicaldev/image/fetch/s--YVMmwJ8G--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/gdnd30dncbus7bnhz717.png)



## Healofy Architecture: Behind the scenes of 4 million daily active users 

## Introduction

Firstly, a word about Healofy app

Healofy is a ***women-only*** social networking platform, initially targeting pregnant women, for pregnancy baby care and pregnancy tips etc. It is a pregnancy and parenting platform for Indian mothers. Now, it has evolved into a social network, generating feeds, where users get to see health tips, nutrition, fitness, fashion, beauty, food recipes, relationship advice etc. 

It has almost become an go-to app for women.

In the first part of this article - I am sharing my system design perspective/ thoughts. If you are not interested, you can directly skip to the second part of the article which contains architecture details (web-scraped & summarized from internet/ videos ).

Secondly, a word about statistics of Healofy app.  

According to [this](https://www.prnewswire.com/in/news-releases/healofy-india-s-largest-women-social-network-set-for-rapid-growth-aims-to-connect-100-million-women-online-863398646.html#:~:text=With%203%20million%20downloads%2C%20and,mothers%2C%20and%20life%20beyond%20that.) , it says, around half a million DAUs around Nov 2019 and according to [this](https://www.youtube.com/watch?v=11kInfzWUFo), we can find that , by Aug 2020, it is around 4 million daily active users.

### **<u>PART-1</u>** - System Design Perspectives

Let us think for a moment : what are the different **functional requirements** from this app?

( If we have used Facebook , it is easy to come up with the basic requirements as listed below, since this is also a social network)

1. Users should be able to  create a post - textual / upload photos/ videos
2. Users should be able to view a newsfeed
3. Users should be able to connect with others (follow / friend request etc.)
4. Users could create their pages, focusing on their personal brand 
5. For all active users, whenever there are new posts created by other users, the system should be able to append to the newsfeed for these active users.
6. Since the app is going to deal with first time internet users, regional languages should be supported, both in the user interface as well as in the content shared by users, in general.
7. Users should be able to form groups/ communities and chat / post within the group / community and the interaction activity should be visible near-realtime to other users of the group/ community.
8. Users should be able to transact from the social network, so integration of a reliable payment system should be present.

What would be the **non-functional requirements** from this app? (in terms of latency, scalability etc)

1. The system should be able to generate the newsfeed for any given active user, within a maximum of 2 seconds
2. When a user publishes a post, this post should make it to the newsfeed of other users, within a maximum of 5 seconds.

**Capacity considerations**

1. Storage: Let us assume, that, each user posts some content. And for simplicity sake, let us assume that, each user is posting some content of size 10 KB. 
   1. Let us also, assume that, for each user, average post length in a newsfeed is around 10. So, totaling 10 x 10 KB = 100 KB, for 10 posts.
   2. Let us go one step further and also assume that, for quick fetching purpose, around 100 posts need to be stored in memory, So, totalling 100 x 10 KB = 1000 KB for each user.
   3. Now, there are atleast 4 million active users and for each user 1000 KB is required to be stored in memory, so totalling 1000 KB x 4,000,000 = 1 MB x 4, 000, 000 = 4000 MB x 1000 = 4 GB x 1000 = 4000 GB = 4 TB, so 4 TB memory required to store posts.
   4. If a server can hold 40 GB memory, then backend will require atleast 100 server instances, in order, to maintain top 100 posts in memory for all active users.
2. Traffic: Let us assume that, on an average, one user has 100 friends and follows 50 pages. Remember that, there are atleast 4 million active users. Let us also assume that, on an average, each user is accessing their timeline newsfeed for atleast 5 times a day, so there will be 20 million news feed requests per day , or, approximately 5556 newsfeed requests per second.

### <u>PART-2</u> - Architectural Details



Following are the kind of problems they need to solve.

1. Scale
2. Localization -Regional languages - the application should be available in native language format, so that, the penetration is large.
3. Safe environment of trust
4. Helping women earn on the platform
5. Catering to First time internet users —>
   1. So, it requires that app should be simple which means the front end should be simple for the users to use, so, backend needs to address lot of things.

**Facts about usage of Google Cloud Services:**

Scalability of Google Cloud powering Healofy:

1. Userbase in first year - 2017 - 1.5 lakh
2. AppEngine — >
   1. 4 million sessions on Healofy everyday (to get a perspective: it is equivalent to los angeles population https://worldpopulationreview.com/us-cities/los-angeles-ca-population )
   2. At any point in time, 1000 instances up and running
   3. cache hits daily — 100 billion cache hits
      1. 1.2 billion are read operations
      2. 300 million are write operations
3. Cloud DataStore
   1. is the primary storage for Healofy
4. FireStore
   1. Chat groups - women are connected ( 1000 people - 50,000 people )
   2. 1 million concurrent users at a point is supported by Firestore
5. Cloud Functions
   1. on the fly google transliteration - millions of words are contextually transliterated daily.
   2. Google AutoML
      1. Categorizing the content, on the platform itself automatically. (some of the tasks cannot be done manually at all.)
   3. Google Vision
      1. Categorize video / image automatically
6. Future Vision of the product -
   1. Personalization & Commerce
   2. Live streaming
   
   
   Originally published at https://gansai.blogspot.com/2020/08/healofy-architecture-behind-scenes-of-4.html

