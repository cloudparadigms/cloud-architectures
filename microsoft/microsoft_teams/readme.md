

| Entity                              | EntityValue |      |
| ----------------------------------- | ----------- | ---- |
| No. of organizations using MS Teams | 500,000     |      |
| Total no. of daily active users     | 75,000,000     |      |
|                                     |             |      |



## MICROSOFT TEAMS Architecture: Behind scenes of 75 Million Daily Active Users


## Introduction

Firstly, a word about statistics:

75 Million Daily Active Users ( DAUs ) - this is as per April 2020 statistics. 

Let's also remember that DAU represents active users - The actual total no. of users might be far greater than this. 

To give a perspective of this data of 75 million Daily active users, imagine all the people belonging to TamilNadu state of India, is using MS teams - actively & daily. ([Tamil Nadu population - close to 77 million, by 31st May 2020 ](http://www.populationu.com/in/tamil-nadu-population#:~:text=Tamil Nadu population in 2020,the capital of Tamil nadu.)). 

If someone is interested in source of 75 million DAU - you can refer to [below tweet](https://twitter.com/chriscapossela/status/1255617866935595008) from Microsoft CMO.

(Ofcourse, we can factor out weekends, assuming that, this figure could drop by almost half, considering that most of the companies have weekends off  )

As of today ( as I write or as you read ), it would have grown even more.

In the first part of this article - I am sharing my system design perspective/ thoughts. If you are not interested, you can directly skip to the second part of the article which contains architecture details (web-scraped & summarized from internet/ videos ).

### **<u>PART-1</u>** - System Design Perspectives

Ever since I started using MSTeams, I was curious behind the architecture which powers this product,

Fortunately, Microsoft has done multiple sessions on this.

I had some doubts regarding MSTeams, Some of them are below (I guess, I am speaking for all the curious users of MS Teams here):

- How is the message delivered ?
- Does it have reply chains ? ( Similar to Facebook Comments/ Reddit Comments ) No, It is a straight line of chats.
- What is UI framework used ? Electron ( I guess, but, now it is React-Native ).
- (Most importantly) What are the different Azure Cloud Services which is powering MS Teams?
- What are the dependencies in Microsoft Office 365 ? (Office suite online)
  - As I noted, Word/ PPT is embeddable in Teams? (From above questions, I am also assuming that a well-defined contracts along with well-defined standards are maintained by different services provided within Microsoft ecosystem )
    - For example, what are the different REST APIs ? We can also assume that there are lots of microservices.
      - Another question - How many are java-backed / c#-backed / nodejs-backed - I guess, this does not matter?
        - Probably performance matters - so choice of language
        - Probably team members who own that service - so choice of their language - they might be well versed with.
        - Probably all that matters is it exposes rest services - which is well-defined & well-designed. It need not be just rest services based solutions. 
- Are there any REST APIs which Microsoft Teams is using? Yes, if I am not wrong, it is Microsoft Graph API.

***From a system design perspective:***

1. Who are the **users** of this product?

   1. For enterprise usage:
      1. Employees of an organization
      2. Administrators - who onboard the application to the organization and manage the product, for resolving internal queries.
   2. For personal usage:
      1. Single user / Family ?

2. **Functional** requirements:

   1. Users should be able to send a message to another user
   2. Users should be able to interact with other users within context of a 'Team'
   3. Users should be able to search for other users within the organization
   4. Users should be able to upload / download / preview files -- images/ text files / videos/ audio
   5. Users should be able to initiate a meeting

3. **Non-functional** requirements:

   1. **scalability** :
      1. MS teams should be scalable to accomodate a organization of varying sizes ( Here , I am curious as to which is the smallest organization (not personal usage) which MS Teams has onboarded and which is the largest organization which MS Teams is supporting? )
   2. **downtime** : 
      1. There are lot of scenarios to be considered here:
         1. What if there is a user chatting and sending some important message and MS Teams backend is down ?
            1. Probably the MS Teams is resilient enough. 
         2. What if there is a live meeting, amongst multiple participants - assume that the network is up for all participants, but MS Teams backend goes down?
            1. Teams might be recording the meeting as well. So, we can imagine that Microsoft Video Stream service is being used.
   3. **upgrade** :
      1.  How is MS Teams upgrading itself, without affecting the continuous streams of chats/ file uploads/ ongoing meetings.
   4. **performance**:
      1. What is the performance guarantee provided by MS Teams, for example, in terms of latency ? For example, when a user searches for some content - a specific keyword - what is the performance guarantee for this query, at the server side? Here, the delivery aspect depends on the network owned by the organization - connectivity with Microsoft Azure infrastructure. But the query-service aspect depends on the algorithms deployed & running internally within Microsoft infrastructure.
   5. **availability / SLA**:
      1. availability of multiple services powering MS Teams. Here it is safe to assume that, since multiple services are powering MS teams, if one service goes down, it does not make sense if all other services are up and running. Again here, we can also safely assume that, probably the user/ the team - is not audience of all the services simultaneously.
   6. **reliability**:
      1. MS Teams should be highly reliable - in the sense - any uploaded file should never be lost
         1. I guess , this guarantee is ensured by Onedrive for business
            1. I am still unsure as to what kind of database/ datastore powers Onedrive - which provides this guarantee

4. **Design Considerations**:

   1. **Read-heavy or Write-heavy**: If we consider this aspect, MS-teams could be designed to be write-heavy. Probably the number of writes are far greater than the number of reads. Because, users are constantly writing to the channels / teams. More often than not, users generally read the latest posts in the channel. Very rarely, users scroll up the feed - which would require multiple reads from the backend. Here, cache could be considered as an option - why should there be a network latency introduced, when we can store content of the device itself.
   2. **Cache** : Here, again, we should be able to determine if its a smartphone device or a desktop device. With regard to desktop device, the amount of storage is far greater than amount of storage provided by smartphones. So, appropriate decision could be made. For example, there might be more network latency introduced by MS-teams on smartphone, than MS teams on desktop device.
   3. **Synchronization**: Suppose there is a user who has installed MS teams on smartphone as well as desktop device

5. **Capacity Estimation & Constraints**:

   1. Let's assume the current no. of users: DAU : 75 million total daily active users. (75,000,000)

   2. Let's also make a fair assumption that, each users posts around 100 text messages () per day. And each message is around 100 bytes. So, 100 x 100 = 10 kb - So, each users contributes around 10 kb.

   3. Overall, 75 million DAU, will contribute around for text messages alone - 

      1. 75,000,000 x 10 kb = 750 megabytes per day ( actually this is very less, this can fit within most pendrives - my assumption is not realistic, but let's continue with this unfair assumption 😊 . Definitely, in terms of enterprise-cloud-thinking - this is not an average enterprise workload - the amount of computing & storage required, inorder to run a workload might vary from usecase to usecase. food for thought - what are the compute-intensive workloads in cloud and what are the storage-intensive workloads in cloud )

   4. So, total space required for text messages per day, for all 75 million DAU, amounts to just 750 MB.

   5. Total space required for 1 day of text messages from all 75 million DAU is 75M * 10 KB = 750 MB.

   6. Total space required for 5 years = 750 MB x 5 YEARS x 365 DAYS = 13,68,750 MB or approx 1369 GB. or approx 1.4 TB.

      1. This can easily fit in hard-disks and with regard to cloud storage offered by Microsoft infrastructure, it is absolutely minimal. ( food for thought: What is the overall storage offered by Microsoft Cloud infrastructure ? )

      2. (food for thought: What if we also factor in - audio / video (streamed content) - how much storage would be required approximately?)

      3. (***Assumption !!!, not a fact***) If we consider below table related to conversion of bytes for disk storage, probably Microsoft Cloud Storage should hold atleast 1000 petabytes overall. (for 5 years, storing text messages)

         1. The fair assumption is that, the average organization would require around 1.4 terabytes.

         2. As of March 19, 2019, there were 500,000 organizations. So, 500,000 x 1.4 TB = 700,000 TB = 700 PB ~ 1000 PB

         3. ##### Disk Storage (thanks to https://pradeepedwin.wordpress.com/mb-gb-tb-pb-eb-zb-yb-bb/)

            · 1 Bit = Binary Digit
            · 8 Bits = 1 Byte
            · 1000 Bytes = 1 Kilobyte
            · 1000 Kilobytes = 1 Megabyte
            · 1000 Megabytes = 1 Gigabyte
            · 1000 Gigabytes = 1 Terabyte
            · 1000 Terabytes = 1 Petabyte
            · 1000 Petabytes = 1 Exabyte
            · 1000 Exabytes = 1 Zettabyte
            · 1000 Zettabytes = 1 Yottabyte
            · 1000 Yottabytes = 1 Brontobyte
            · 1000 Brontobytes = 1 Geopbyte



***End of system design perspective***

### **<u>PART 2</u>** - Architecture of MS Teams - High Level Client & Services Architecture



### Teams logical architecture





[![img](https://gxcuf89792.i.lithium.com/t5/image/serverpage/image-id/54818i713FD46F5B191197/image-size/large?v=1.0&px=999)](https://draft.blogger.com/blog/post/edit/7872816081050679724/3594879909122532238#)





Team is backed by Office 365 Groups.

The membership of that team -- add members, remove members.

That team has apps,

Team is backed by instance of Sharepoint. Sharepoint membership is backed by Office 365 group. Files belonging to 

Teams can have many channels. They could be topics.

Each channel has a folder in Sharepoint where files are stored

Each Channel has Tabs. Microsoft and 3rd party tabs.

Within Channel, there are messages, reply chain. Each Message can have media associated with them.

Chat on other hand uses OneDrive Business for storing files.

It also has tabs and apps. But it is a smaller set, relative to Teams. It does not leverage reply chain. It is a straight line. 

We have 1-1 chat or group chat.

Participants in a chat can go upto 250.

Meetings,

Calling,

Voice mail associated with that Calling.

Activity Feed - In a Team, when someone mentions/ likes/ replies, copy of that, and put in personal stream to you.



### MS Teams - 10000 foot view



![image-20200808200533782](https://1.bp.blogspot.com/-e4O13heeOUE/Xy7_v2ZY23I/AAAAAAAAvOY/cW0kFy-zzzwWfLzG-i6pOPUksnhnXfN2QCLcBGAsYHQ/d/image-20200808200533782.png)

Above is the 10,000 foot view of high level client & services architecture powering MS Teams. These are the building blocks of MS Teams. MS Teams leverages Office 365 Platform and its services. MS Teams has its own services which powers the product. Intelligent Communications Cloud which was previously Skype Infrastructure. (so, we can assume that the acquisition of Skype has played some role). And महत्वपूर्ण बात ये है कि , முக்கியமான விஷயம், ముఖ్యమైన విషయం, ಮುಖ್ಯ ವಿಷಯವೆಂದರೆ - Azure Cloud Services.



### Teams Client Architecture

![image-20200808200846567](https://1.bp.blogspot.com/-sudCHxb4Tk8/Xy7_8Bf3qLI/AAAAAAAAvOc/7YvQzzujLB8FtEGJQHFFytRa_h7Xp6r9wCLcBGAsYHQ/d/image-20200808200846567.png)

Above represents the Client Architecture. We have the desktop client & smartphone client - this can be either iPhone/ iPad or Android. From the desktop mode, we could have clients coming in either from Web/ Windows Desktop or Mac Desktop modes.

The clients are built for agility (fast moving ) and for auto-updates. 

### Release Rings

Detouring to a concept called 'Release Rings' - that is the updates of the product are well-tested & managed, prior to public-rollout. 

Below is an illustration related to Office 365 product release ring. The same applies to MS Teams as well.

![image-20200808202100468](https://1.bp.blogspot.com/-nLv_0hcjJrQ/Xy8AGInEN_I/AAAAAAAAvOk/AmJvqkXadV0u4wj8N3sbNLcLxvJe9V11gCLcBGAsYHQ/d/image-20200808202100468.png)

​				courtesy: https://www.itprotoday.com/office-365/office-365-adopts-rings-updates-testing-public-release

- **Ring 0** – The feature teams of engineers who build and test proposed changes.
- **Ring 1** – The Office 365 team takes a test drive.
- **Ring 2** – All Microsoft employees (dog-fooding).
- **Ring 3** – First Release customers. Those brave, opt-in souls who don't mind providing feedback and bug reports.
- **Ring 4** – Everyone! The official worldwide rollout.

We can safely assume that some of the features are actually present in our own systems, but they are turned off, until it is rolled out to public.

Back to Client Architecture:

1. It is based on HTML/ CSS.
2. Code base is moving slowly from Angular to React
3. It is based on lots of open source libraries like: jquery, lodash
4. The client is completely written in Typescript, and transpiled to Javascript.
5. Desktop version uses Node,  meeting features written in C++ for Windows, Objective-C for Mac.
6. Electron - web app, wrapping in an executable .
7. Mobile - It uses React Native. 



### What are Teams Services ?

![image-20200808235039083](https://1.bp.blogspot.com/-SbdA6Tigg8s/Xy8APUTrqsI/AAAAAAAAvOs/IvW8wngmoFMdLHyxxXPeHNxRq1dPoEjBQCLcBGAsYHQ/d/image-20200808235039083.png)



Teams Services:

1. Organized into Identity and diff components as presented as in above diagram.
2. It is now a collection of microservices, you can independently scale up.



### What is Intelligent Communications Cloud ?

![image-20200808235242152](https://1.bp.blogspot.com/-Xcq2TzeCO-c/Xy8Acl9uEdI/AAAAAAAAvO4/_7sJ9AL5OX0dyuWuqxtkpvQd_bbh1ZuLwCLcBGAsYHQ/d/image-20200808235242152.png)



1. This is where the PSTN integration code. It uses Skype stack.

2. Presence services.

3. Messaging stack - serves messaging chat, media, search, URL preview - crawl the link to show the preview.

4. Config/ Experimentation - useful for A/B testing or release rings.

   
###Teams dependency on Office 365

   ![image-20200808235555349](https://1.bp.blogspot.com/-B5ry0AWA3jM/Xy8AjdOyn2I/AAAAAAAAvPA/RakOruImFA45goB-jDKyDRGwA-JZPxTtACLcBGAsYHQ/d/image-20200808235555349.png)

   



1. On the platform side, we can see different services on which Teams is based on. For example, Exchange is for email, Calendar 

2. Sharepoint, Stream - for voice recording, video recording.

3. PowerBI - for analytics - internally.

4. Teams makes itself as HUB of Office 365 - makes it greater than sum of all its parts.

5. Teams uses most of Azure. ( Keyvault, Blob storage ) - most of complexity is offloaded to Azure, resiliency, data residency, massive scale of Azure , data compliance standards relies on Azure.

   

   ### Teams dependency on Azure Services

   ![image-20200809000049509](https://1.bp.blogspot.com/-8LAaEO4xlrs/Xy8AqWxXNmI/AAAAAAAAvPE/9Ra2e0gu-4ABYIUUXYERGpyHRw4gDqEgACLcBGAsYHQ/d/image-20200809000049509.png)



### Deployment Patterns and data residency

![image-20200809000137062](https://1.bp.blogspot.com/-93XwcxVkpM0/Xy8AwvVs8HI/AAAAAAAAvPQ/h0xlq3eeM0oDFfOV9KpSuhmXIaFAs09LACLcBGAsYHQ/d/image-20200809000137062.png)





Above image shows about deployment patterns and data residency requirements.

If a customer's tenant is in India, for example, we store customer data at rest in that region - for example: chats, conversations, images, etc stored in that region.



### High level architecture - 1 level deeper

![image-20200809000324447](https://1.bp.blogspot.com/-GlFib4ZCWpA/Xy8A2tvuG_I/AAAAAAAAvPY/UuUdETIKp-gA36yV4eNdps24BvIWaVVtQCLcBGAsYHQ/d/image-20200809000324447.png)



1. Graph Api - is also a client - goal is to expose functionality. 
2. Teams client - talks to the services directly. WAC - web application companions. (word, excel,..)
3. Connectors - webhooks - Azure devops for example.



### Messaging Flow

![image-20200809000657991](https://1.bp.blogspot.com/-apZblSUSDFA/Xy8A8z7tm_I/AAAAAAAAvPg/Q3GGSL2kSrcXyS7XUN_jWHYDmvAEOcJ1ACLcBGAsYHQ/d/image-20200809000657991.png)



1. Messaging Data architecture:
   1. Teams client talks to no. of services - media service, real time notifications, 
   2. Chat services - core data structure - Message Thread, members of chat are stored in a roster.
   3. Pub-Sub
2. Message Sync: after reconnect, it gets the data.

### What happens when you post a message?

![image-20200809000953269](https://1.bp.blogspot.com/-M4O9c0GWS2g/Xy8BD5zpNkI/AAAAAAAAvPo/WZeiM6N9J_sV5CLnCBFfm7UJCUFxwcAswCLcBGAsYHQ/d/image-20200809000953269.png)



1. UserA - on device 1 - posts a message - talk to chat service.
2. UserB - just logged in - sync should happen.
3. UserB on device 3 - already logged in --> syncing via long poll -- real time notifications.
4. Lot of notifications happen - Exchange gets notified, Search Index is updated.. etc for all subscribers.
5. Smartphone device get notified.



### Conversation Storage

![image-20200809001313182](https://1.bp.blogspot.com/-BUiwaVb2R8s/Xy8BJ8dd3wI/AAAAAAAAvPw/WyEWOtu5A24kPS0ryQ04NEV1Gk50bugGQCLcBGAsYHQ/d/image-20200809001313182.png)



### File Storage

![image-20200809001409621](https://1.bp.blogspot.com/-8xZs6HVgCoc/Xy8BPmXHxfI/AAAAAAAAvP4/dmBQDKZUpGkvCzfnR7zXmHV0ACJLVmNRgCLcBGAsYHQ/d/image-20200809001409621.png)



1. Team is built on top of groups - Sharepoint is leveraged internally.



### Where things are stored ?

![image-20200809001515233](https://1.bp.blogspot.com/-_c0DjUSHAi8/Xy8BV7oUtaI/AAAAAAAAvQA/EM--vVPaOBITg9_iw-iU-iWxpWIag0OrQCLcBGAsYHQ/d/image-20200809001515233.png)



Telemetry - record anything what user does, but strips privacy / identification data.



### Calendar Architecture

![image-20200809001624480](https://1.bp.blogspot.com/-2Dy8RyFeHls/Xy8Bb6_Js8I/AAAAAAAAvQE/YhZWOJE-EbMOa43UOX4x3PaJeCeuBKY-QCLcBGAsYHQ/d/image-20200809001624480.png)

1. When scheduling meeting, free/busy info is required, for this, there is a service.



### Meetings Flow

![image-20200809001729246](https://1.bp.blogspot.com/--yBK3mFVcu0/Xy8BiKjethI/AAAAAAAAvQM/_A9Fs4gn31gVbyyDqncMKtIFF4Lu6EI4QCLcBGAsYHQ/d/image-20200809001729246.png)



1. In Teams, meetings is a group chat, with extra properties.
2. Meeting is stored in exchange.
3. Voice/ Video goes to Media mixer service.

### Messaging and Presence Interoperability

![image-20200809002046334](https://1.bp.blogspot.com/-ntRNjKfxv2g/Xy8BqnZ31sI/AAAAAAAAvQU/amdo7FrAUIIk96bqP5m0oDPw6cuwNFVbgCLcBGAsYHQ/d/image-20200809002046334.png)



### Meeting recording flow

![image-20200809002147757](https://1.bp.blogspot.com/-3dGaj68ldbU/Xy8Bqv0dLyI/AAAAAAAAvQc/VISVB5G0ChQVrfkhODlBuCKot366jqF2gCLcBGAsYHQ/d/image-20200809002147757.png)





### Client Calling stacks

![image-20200809002451036](https://1.bp.blogspot.com/-POyGbx_V0hw/Xy8Bqq-wzVI/AAAAAAAAvQY/iK-ZANKdRaAfDgjwSdkgs4DLDAZdftfogCLcBGAsYHQ/d/image-20200809002451036.png)



1. The calling service from the client, goes to the typescript api - called ICall API, which inturn talks to SlimCore - which is a native code.
2. The SlimCore native , in turn talks to underlying services.



### Calling Flow



![image-20200809010041800](https://1.bp.blogspot.com/-u6U7dImpdnw/Xy8B0ZOSl2I/AAAAAAAAvQo/QagUya5LjiUmeWsgAKivyMWZMoMLTU9ZgCLcBGAsYHQ/d/image-20200809010041800.png)



## Data Compliance

[![img](https://gxcuf89792.i.lithium.com/t5/image/serverpage/image-id/54820i670ED5E0FFC3AFE1/image-size/large?v=1.0&px=999)](https://draft.blogger.com/blog/post/edit/7872816081050679724/3594879909122532238#)

### Compliance Boundary

============================

Promises - Security, Privacy, Compliance

All data stored, is encrypted at rest, at boundary.

What are various ways data is coming in:

Many organizations want to lock things down.

Connectors - data coming in. eg: JIRA, work items..

Email channel - coming in from anywhere.

Giphy - query - sent to Giphy org. Celebrate, fantastic..

 be aware that going outside of org.

 

Push Notifications - to notify mobile client.. outside of 

Microsoft purview.

URL Preview - leaving compliance boundary. 

there are some orgs want to tighten.

Tabs, Apps/ Bots - bot hosted anywhere in world.

Graph API- access data - can be controlled access.

Calling - can go to anywhere.

Users - can be anywhere.

Anonmymous user joining - client joining 

Federation communication - one tenant to another tenant.

 -- selected set of tenants.. or completely lock it down..

Communicaton -- customers, partners..





Please note that, many of the above parts might have become obsolete, as software is ever evolving 😊

So, the next time, you use MS Teams, you know where the bits are flowing 😊🤖

Originally published at: https://gansai.blogspot.com/2020/03/microsoft-teams-architecture.html



