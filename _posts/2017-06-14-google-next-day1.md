---
layout: post
title: "Google Next Tokyo 2017 Day 1"
description: "Some notes taken on some sessions I joined in day 1"
tags: [note]
---

## Machine Learning Intro

- #### Deepmind new technology: WaveNet [Link](https://deepmind.com/blog/wavenet-generative-model-raw-audio/)
  * more human alike voices


- #### Gmail Smart Reply
  * content understanding, create response automatically
  * occupies 20% of mail
  * maybe like linkedin's auto message response?


- #### Google DataCenter
  * Remove 40% of cost on cooling equipments, PUE 15% improvement

- #### GCP machine learning services
  * Self learning
    - TensorFlow
    - Cloud Machine Learning Engine
  *  API Service when finished learning
    - [Cloud Vision API](https://cloud.google.com/vision)
    - Could Speech API
    - Cloud Natural Language
      * no need knowleddge base in AI
      * NL analyze to catagorize text & segments

- #### Example from Adtech studio
  * Sugio Tatsuki (DSP/DMP/代理店, AI Lab)
    - Versus using [Vision API](https://cloud.google.com/video-intelligence)
      * animate ad increasing
      * get out basic elements/content/meaning using cloud vision API
      * YouTube Brand Lift Survey (Previous use human to import)
        - hard to tag animate (before)
        - PDCA speedup / cost, human error reduction (after)


- #### Example from Kewpie
  * using Cloud Machine Learning Engine
    Find product with defect

- #### Example from オークネット

- #### Example from アクサ保険
  * Predict large accident happening, accuracy 78.3%
  * Using Neural Network

## Hands on Session Part - Financial Group from 三井住友
SMFG's AI example - TensorFlow Service Introduction
(Not Recommend, this is only a self-advertisement session)

- JSOL Corp. Japan Research Institute, 1969~
  * Serve many stock companies and financial related customers
  * Business problem solving, Data Science, System engineering
  * Prevent Credit Card Fraud
    - Use human for double check the result
  * Detection of degradation
    - Financial Report analysis
      - Predict if the changes in budget is possible.
  * Cooprate with Peers.Lab
    - SVM about 60~70% accuracy, Deep Learning about 90%
  * Data creation, Data Upload , ML data refine, ML input data creation, ML Model creation, Accuracy evaluation
    Peers.Lab's service includes all steps after Data Creation
  * ML: AD/CNN, setting: UI, Data statistics: min/max/avg/Standard Deviation/learning curve
  * Price about 2\*gcloud price

## Google Datascientist
Takashi J. Ozaki, Ph.D., Ad Department
- Agenda
  * machine learning how it works
  * machine learning 8 steps
    - What reason do you want to introduce machine learning:
      automatic ad delivery, ec site, SNS
      want to automatically finish frequent jobs
      CPA to max
    - How do you collect data
      choose only necessary data with relationship
    - preprocessing for data
      preprocessing could occupy 80% of cost
    - model learning
      linear & non-linear (SVM, correlation...)
      not almighty
    - Training
    - general ability
      Give better accuracy for unknown data
      overfitting problem
    - Check
      A/B test, pre/post test
      ROI is also important
    - improve サイクル
      Changes in environment
  * GCP x ML for business
    - find out new user
    - image recognition, creation
    - document recoginition, catgorize
    - recommend data
    - strange data detection
  * GCP x ML architecture
    - Google Analytics 360 + BigQuery -> Measurement + attribute
  * Live demo
    - With fewer data, MLP is better than Neural Network (in demo for 100 steps, 73% vs 97%)
  * GMO Ad Marketing
    Refer to GCP Japan Blog


## Cloud Spanner Introduction
[Product information](https://cloud.google.com/spanner/)
- Agenda
  * The history of spanner
    - Why did they build?
      2005 use sharded mysql for ad department
      * horizontally scale possible
      * global consistency (ACID)
      * not accept down-time
    - What is Cloud Spanner
      with global scale, use sql to work with
      cross-region, run almost 10 years in google
      almost all services in google use it
    - Ability
      Schema, SQLm consistency, availability, scalability, replication
      all automatically or strong!
      - No-sql has low consistency (eventually)
      - ANSI 2011 SQL
      - Java, python, go, Node.js, soon php, ruby, C#, JDBC driver support
      - log tracing, id management, encrypted
      (You can seamlessly integrate it to your services!)
    - Workload category
      - strong transaction consistency requirement
      - scale with mysql, posgressql and any other relational requirement
      - keep the state of web application
      - consolidate all the different databases using a single spanner
    - Cloud instances
      * split by zones, each zones have multiple DB
      * auto replica setting and update replica
      * elect one as master, for schema up-to-date checking
      * child table support - structured table for speed
      * no downtime on attribute integration/update
      * auto load balance, but with longer query time at 
        the first step on establish connection before addicted 
        to a single instance, the speed is comparable to cloudsql
      * support asia-east1, us-east, europe
      * optimizer hints using explanation feature
    - API
      * with staleness settings, we could cache the data a little bit
        for speedup without query twice
      * RW transaction and Read-Only transaction
      * Use Mutation as ORM
      * don't use tiemstamp as primary key, use getUUID API to gain
        performance, otherwise it will rely on a node for computation
      * using run function for transaction re-executation when failed

## API.AI & Cloud Speech to provide Chatting User Experiences
------------------------------------------------------------
Originally a 2-hour speech in SF, but squeezed to 40min in Tokyo.
API.AI website [link](https://api.ai/)

- Pros
  * support 14 languages
  * no coding needed
  * highly support conversation
  * end-to-end suit
  * tightly connected to many platforms (slack, facebook...)
- Live Demo
  all operations and testing could do in web console.
  but you still could upload data in batches for training.
  This is more of the NLP tech, if some words/sentences fail to parse, you could find it out in logs and give it the correct meaning.
