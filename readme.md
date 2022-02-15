### Software Design Documentation

### Factory 2.0
Date: Feb 14, 2022

#### Introduction
**Purpose**
This software design document describes the structure of the software and how it will be built and executed. The file provides technical details and a description of all methods and technologies.  

**Scope**
The software design document’s scope sets the requirements for the software, helping the team and the stakeholders summarize the characteristics of the desired product. Here, parties state which features are essential to achieve business objectives and user experience goals. In particular, the document is focused on describing the essential functionality and critical architectural components. 

**Audience**
This document will be created and used by the development team, project manager, and the client. The process of making changes to the software design document should be discussed with all participants. All stakeholders are free to refer to SDD at any stage of the project. 

#### System Overview

```mermaid
    flowchart TB;
   A[Customizer]-- order -->B[Customizer Api];
   B[Customizer Api]-- store -->C[(Api DB)];
   B[Customizer Api]-- created -->D[QX7/Dealer];
   D[QX7/Dealer]-- store -->E[(QX7 DB)];
   D[QX7/Dealer]-- approved -->F[Factory];
   F[Factory]-- store -->G[(Factory DB)];
```

#### Design Considerations
**Automation**
**Backward Compatibility**
**Mobile Optimized**
**Performance**
#### System Architecture and Architecture Design
- System Architecture Diagram

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
    
- Software Architecture Diagram

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
- Sequence Diagram   
``` mermaid
    sequenceDiagram
    autonumber
    User->>Login Page: Login;
    Note left of User: User Login Sequence;
    Login Page->>Validation: Login(User and Password);
    Validation-->>Aws Cognito:Verify(User and Password);
    Aws Cognito-->>Validation:User verified;
    Validation-->>User Model: Check If User Exist;
    User Model-->>Validation:User Found;
    Validation-->>Login Page:Login Success;
    Aws Cognito-->>Validation:Unauthorized;
    User Model-->>Validation:User Not Found;
    Validation-->>Login Page: Login Failed;
```
---
```mermaid
    sequenceDiagram
    autonumber
    User->>Factory Page:access page;
    User->>Factory Setup Page:setup factory;
    Factory Setup Page-->>Factory API:get master processes;
    Factory API->>Factory Setup Page:show master processes;
    Factory Setup Page-->>Factory API:create and store new factory process;
    Factory API-->>Factory Page:display factory processes;
    Note left of User:Factory Page Access Sequence;
    Factory Page->>Factory API:verify user access token;
    Factory API-->>Factory Page:user token verified;
    Factory API-->>Factory Page:display factory information;
    User->>Factory Page:edit factory;
    Factory Page->>Factory API:get factory information;
    Factory API-->>Factory Page:show factory information;
    User->>Factory Page:update factory information;
    Factory Page->>Factory API:store factory information;
    Factory API-->>Factory Page:save success;
    Factory Page-->>User:show save success;
```
#### Design Specifications
- Business Requirements
- Database Design
- User Interface Design
