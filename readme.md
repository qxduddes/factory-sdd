### Software Design Documentation

### Factory 2.0
<small>Date: Feb 14, 2022</small><br>
<small>Author: Agustin Depasupil</small><br><br><br>

> #### 1. Introduction
**1.1 Purpose**<br>
This software design document describes the structure of the software and how it will be built and executed. The file provides technical details and a description of all methods and technologies.  

**1.2 Scope**<br>
The software design document’s scope sets the requirements for the software, helping the team and the stakeholders summarize the characteristics of the desired product. Here, parties state which features are essential to achieve business objectives and user experience goals. In particular, the document is focused on describing the essential functionality and critical architectural components. 

**1.3 Audience**<br>
This document will be created and used by the development team, project manager, and the client. The process of making changes to the software design document should be discussed with all participants. All stakeholders are free to refer to SDD at any stage of the project. 

> #### 2. System Overview

```mermaid
    flowchart TB;
   A[Customizer]-- order -->B[Customizer Api];
   B[Customizer Api]-- store -->C[(Api DB)];
   B[Customizer Api]-- created -->D[QX7/Dealer];
   D[QX7/Dealer]-- store -->E[(QX7 DB)];
   D[QX7/Dealer]-- approved -->F[Factory];
   F[Factory]-- store -->G[(Factory DB)];
```

> #### 3. Design Considerations
- **3.1 Automated**<br>
- **3.2 Resiliency**<br>
- **3.3 Mobile Optimized**<br>
- **3.4 Performance**<br>
> #### 4. System Architecture and Architecture Design
- 4.1 System Architecture Diagram

![PubSubArch](pubsub-architecture.png)

#### Order Flow
```mermaid
    flowchart LR;
    A[Customizer]-- new order -->B[Customizer API];
    B[Customizer API]-- status -->A[Customizer];
    B[Customizer API]-- store -->id1[(API DB)];
    B[Customizer API]-- created -->C[QX7/Dealer Page];
    C[QX7/Dealer Page]-- store -->id3[(QX7 DB)];
    C[QX7/Dealer Page]-- received -->B[Customizer API];
    B[Customizer API]-- received -->A[Customizer];
    C[QX7/Dealer Page]-- approved -->id2[[Queue Service]];
    id2[[Queue Service]]-- new order -->E[Factory API];
    E[Factory API]-- status -->id2[[Queue Service]];
    E[Factory API]-- reject -->id2[[Queue Service]];
    id2[[Queue Service]]-- foid -->B[Customizer API];
    id2[[Queue Service]]-- status -->B[Customizer API];
    id2[[Queue Service]]-- status -->C[QX7/Dealer Page];
    id2[[Queue Service]]-- reject -->B[Customizer API];
    id2[[Queue Service]]-- reject -->C[QX7/Dealer Page];
    E[Factory API]-- store -->G[(Factory DB)];
    E[Factory API]-- order queue -->H[Factory App];
    H[Factory App]-- reject -->E[Factory API];
    E[Factory API]-- shipping -->F[FEDEX/UPS];
    F[FEDEX/UPS]-- tracking -->E[Factory API];

```
    
- 4.2 Software Architecture Diagram

``` mermaid
    graph LR;
    id1((user))--->a[web browser]--https-->c[factory api];
    id1((user))-->b[mobile browser]--https-->c[factory api];
    subgraph Factory System;
    c[factory api]--read/write-->d[(factory DB)];
    c[factory api]--rejected-->e[queue service];
    c[factory api]-- email status-->f[email service];
    end;
    g[qx7/dealer page]-- order approved-->e[queue service];
    e[queue service]-- order approved -->c[factory api];
    e[queue service]-- order rejected -->g[qx7/dealer page];
    e[queue service]-- order status -->g[qx7/dealer page];
    c[factory api]--shipping-->h[fedex/ups service];
    h[fedex/ups service]--tracking-->c[factory api];

```
- 4.3 Sequence Diagram   
``` mermaid
    sequenceDiagram
    autonumber
    User->>Login Page: Login;
    Note left of User: User Login Sequence;
    Login Page->>Validation: Login(User and Password);
    Validation-->>+Aws Cognito:Verify(User and Password);
    Aws Cognito-->>-Validation:User verified;
    Validation-->>+User Model: Check If User Exist;
    User Model-->>-Validation:User Found;
    Validation-->>Login Page:Login Success;
    Aws Cognito-->>Validation:Unauthorized;
    User Model-->>Validation:User Not Found;
    Validation-->>Login Page: Login Failed;
```
---
```mermaid
    sequenceDiagram
    autonumber
    Note left of User:Factory Page Access Sequence;
    User->>+Factory API:verify user access token;
    Factory API-->>-User:access token verified;
    User->>Factory Page:view page;
    User->>Factory Setup Page:setup factory;
    Factory Setup Page-->>+Factory API:get master processes;
    Factory API->>-Factory Setup Page:show master processes;
    Factory Setup Page-->>+Factory API:create and store new factory process;
    Factory API-->>+Factory Page:display factory processes;
    Factory API-->>-Factory Page:display factory information;
    User->>Factory Page:edit factory;
    Factory Page->>+Factory API:get factory information;
    Factory API-->>-Factory Page:show factory information;
    User->>Factory Page:update factory information;
    Factory Page->>+Factory API:store factory information;
    Factory API-->>-Factory Page:save success;
    Factory Page->>User:show save success;
```
---
```mermaid
    sequenceDiagram
    autonumber
    Note left of User:Factory Setup Sequence;
    User->>+Factory API:verify user access token;
    Factory API-->>-User:access token verified;
    User->>Factory Setup:setup new factory;
    Factory Setup->>+Factory API:get master processes;
    Factory API-->>-Factory Setup: show master processes;
    Factory Setup-->>+User:show setup steps;
    User->>-Factory Setup:select process, create group process,;
    Factory Setup-->>+Factory API:store new factory process;
    Factory API-->>-Factory Setup:get all factory processes;
    Factory Setup->>User:list all factory processes;
```
---
```mermaid
    sequenceDiagram
    autonumber
    Note left of User:Worker Setup Sequence;
    User->>+Factory API:verify user access token;
    Factory API-->>-User:access token verified;
    Factory API-->>+Worker Page:get all workers;
    Worker Page->>-User:view worker list;
    User->>Worker Page:add/update worker;
    Worker Page-->>+Factory API:store worker information;
    Factory API-->>-Worker Page:get all workers;
    Worker Page->>User:list all workers;
```
---
```mermaid
    sequenceDiagram
    autonumber;
    Note left of User:Order Sequence;
    User->>+Factory API:verify user access token;
    Factory API-->>-User:access token verified;
    User->>Order Page:view orders;
    Order Page-->>+Factory API:get all orders;
    Factory API-->>-Order Page:show all orders;
    User->>Order Page:view order information;
    Order Page->>Order Details Page:display order page
    Factory API-->>Order Details Page:show order details to user;
    Order Details Page-->>+Factory API:store/update order details;
    Factory API->>-Order Page:return order page;
    User->>Order Page:assign process;
    Order Page-->>Factory API:generate and store production queue;
    User->>Order Page:select order to view production;
    Production Queue Page-->>+Factory API:get the selected order production queues;
    Factory API-->>-Production Queue Page:retrieve order production queues;
    Order Page->>Production Queue Page:display order production queues;
    User->>Order Page:view production history;
    Production History Page-->>+Factory API:get selected order production history;
    Factory API-->>-Production History Page:retrieve selected order production history;
    Order Page->>Production History Page:display production history page;
    Production History Page->>User:show selected order production history to user;
    User->>Order Page:reject order;
    Order Page->>Reject Page:display reject page;
    User->>Reject Page:input reject reason;
    Reject Page-->>+Factory API:store reject reason;
    Factory API->>Order Page:return to order page;
```
---
```mermaid
    sequenceDiagram
    autonumber
    Note left of User:Production Queue Sequence
    User->>+Factory API:verify user access token;
    Factory API-->>-User:access token verified;
    User->>Production Queue Page:view production queue orders;
    Production Queue Page-->>+Factory API:get all orders;
    Factory API-->>-Production Queue Page:show all orders;
    rect rgb(191, 223, 255)
    opt Accept Order
    User->>Production Queue Page:accept order;
    Production Queue Page-->>+Factory API:set order status to 'Progress' and update order status;
    Factory API-->>-Production Queue Page:change order status to 'Progress';
    end
    end
    rect rgb(204, 255, 204)
    opt Order Complete
    User->>Production Queue Page:order complete;
    Production Queue Page-->>+Factory API:set order status to 'Completed' and update order status;
    Factory API-->>-Production Queue Page:change order status to 'Completed';
    end
    end
    rect rgb(253, 199, 209)
    opt Order Reject
    User->>Production Queue Page:order reject;
    Production Queue Page-->>+Factory API:set order status to 'Rejected' and update order status;
    Factory API-->>-Production Queue Page:change order status to 'Rejected';
    end
    end
```
#### 5. Design Specifications
- 5.1 **Business Requirements**
    - Functional
        1. Able to open the application from different browsers like Brave, Google Chrome, Microsoft Edge and Apple Safari.
        2. The Application user interface should be tablet optimized.
        3. Should be able to easily see the status of the orders like "not started", "on progress", "completed", "cancelled", "rejected".
        4. Should be able to generate and scan unique QRCode for new order queue.
        5. Should be able to scan QRCode to retrieve order information and status.
        6. Should be able to create process flow
        7. Should be able to send email notification on the shipping status
        8. Should be able to show factory analytics in the dashboard.

- 5.2 **Database Design**
---
![Factory Data Model](data-model.png)
- 5.3 **User Interface Design**
