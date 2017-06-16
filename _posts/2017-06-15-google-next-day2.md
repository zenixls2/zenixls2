---
layout: post
title: "Google Next Tokyo 2017 Day 2"
description: "Some notes taken on some sessions I joined in day 2"
tags: [note]
---
All photos taken during these two days have not yet uploaded. Please wait until I have time to do some processing.

- #### Kubernetes
  [Link](https://bitbucket.org/wdenniss/)
  - possible to do cross platform clusters (AWS, GCP, ...)
  - suitable for build up a microarchitecture

- #### Container Engine
  - Manage Kubernetes by google
  - Google is your Site Reliable Engineer
  - Liveness Check - crash location detection, lower downtime
  - easy to scaling up/down
  - Pot is the unit to hold containers
  - node is a VM to run
  - cluster is a collection of machines
  - Demo Session:
  ```bash
  # to watch pod status
  watch kubectl get pods
  # push to docker image in google
  docker --push -- [GOOGLEDOCKER]
  kubectl expose deployment next-demo --port=80 --target-port=3000 --type-LoadBalancer
  # scale up/down
  kubectl scale deploy next-demo --replicas=10
  kubectl delete pods --all
  # no downtime update
  kubectl set image deployment/next-demo next-demo=gcr.io/wdenniss-gke-next.... # use image
  # show history
  kubectl rollout history deployment/next-demo
  # undo
  kubectl rollout undo deployment/next-demo
  # export yaml config
  kubectl get service/net-demo -o yaml --export | tee app.yaml
  # remove all
  kubectl delete deployment,service --all
  # create service
  kubectl apply -f .
  ```

- #### Google Container Registry
  A Circle CI like service, use webhook from bitbucket, github or something else.
  According to the website mentions, it could also integrate with other CI services.
  [Link](https://cloud.google.com/container-registry/)
  * This for continuous integration


- #### BigQuery & Cloud Machine Learning
  The website needs auth [Link](http://104.154.95.26),
  looks like not open to public.
  * Easy to scale, high processing speed
    (400GB/6sec with 100 instances for the cases show during demo),
    no-ops, with SQL
  * suitable for data processing, but not suitable for daily querying
  * UDF support (Eigenvector search)
  * Support Lambda function in javascript for processing
  * Able to import lib from google storage

- #### Cloud Machine Learning Engine
  * Use BigQuery for ETL (map, reduce, filter in js functions)
  * enhance with [HyperTune](https://cloud.google.com/ml-engine/docs/concepts/hyperparameter-tuning-overview)

## BigQuery's new functionalities: Cloud Dataware House
Comparison: [Link](https://news.ycombinator.com/item?id=13916030)
Use with AWS EMR -> [Prestodb](https://prestodb.io).
There's a hosting service for prestoDB: [Link](https://www.qubole.com/products/qubole-data-service/presto-as-a-service/).
Another open-source project, but may not be stable enough for production: [Eventql](https://github.com/eventql/eventql)
- [Dremel](https://research.google.com/pubs/pub36632.html) architecture
  * Master with shard trees
  * scheduler maps ward to shards
- BigQuery Slots
  * a unit of scheduler
  * 1 parallel unit, performance pointer
  * limited CPU and RAM
  * could be start and later canceled
  * unpredictable total amount, it's runtime decided based on the execution time of each slot
  * not neccessary improves performance if there's no enough parrallel possibilities
- High number of distinct values (columns) support
  * use shuffle migration for join/sum up/compare values to speedup
- Shuffle Repartition on shards
  * detect the sharding amount is wrong and change it
- screw Join
  If one shard gets too much data, it would be slow
  * need to fix in the futurer
- Overload the master node
  * ex: use order by might overload the master node (memory not enough)
  * TODO: add LIMIT tag
- data counting methods comparison
  * group by: slow, difficult to incorporate into larger queries
  * count: slow, scalable, exact
  * approx\_count\_distinct: very fast, approximate, within 1% error rate, using [HyperlogLog++](https://en.wikipedia.org/wiki/HyperLogLog) algorithm(support billions of value, less memory)


## Cloud functions for Firebase
[Link](https://g.co.firebase.functions)
Like AWS lambda function + routing
- event driven, js based, an glue to other cloud platforms
- Demo (Hard to take note that fast shown in demo, so some code might be missing. but the concept is the same.)

  Thinking of the case a developer want to create a chat app for international language support and image upload with thumbnail autogen:

    ```javascript
    // this includes database, auth, analytics, and any other functions
    const functions = require('firebase-functions');
    const {spawn} = require('child_process');
    const Translate = require('@google-cloud/translate');
    const translate = Translate({KeyFilename: 'somefilename.json'})

    exports.helloWorld = functions.https.onRequest(
      (res, req) => {res.send(Hello from Firebase"")};
    )
    exports.translate = functions.database.ref('messages/{roomId}/{messageId}/text').onWrite(event => {
      const text = event.data.val();
      const languages = ['ja', 'es'];
      const promises = languages.map(to => {
        return translate.translate(text, {from: 'en', to: to})
          .then(result => {
              const rootRef = event.data.adminRef.root;
              const roomId = event.params.roomId;
              const messageId = event.params.messageId;
              return  ...
            }
          )
        })
    })

    exports.thunbnail = functions.storage.object().onChange(event => {
      const object = event.data;
      const filePath = object.name;
      const fileName = filePaht.splut('/').pop()
      const fileBucket - objct.bucket;
      const bucket - gcs.bucket(fileBucket);
      const tempFilePath = `/tmp/{$fileName}`
      if (!object.contentType.startsWith('image/')) {
        return;
      }
      if (object.resourceState == 'not_exists')
        return
      return bucket.file(filePath.download({destination:tempFilePath}));
      .then(() => {
        return spawn('convert', [tempFilePath], '-thumbnail', '200x200>',
          tempFilePath)
        .then(() => {
          // save back
          return bucket.upload(....)
        })
      })
    })
    ```

  deploy using
  ```bash
  firebase deploy
  ```

  and with cloudwatch like log in web console.
  Remember the google apps needs to restart for testing.
  Most gcloud API could be integrated here.


## Cloud GPU on HPC
-------------------
Notice that this service is quite expensive, but with a bound of graphical software support. I've asked the speaker abount the currency mining topic, and their answer is that currently their internal policy is that they disallow users to do currency mining in their platform.<br>
Agenda
- why GPUs
  * more cores, more bandwidth
  * suitable for linear algebra
  * from AMBER simulation of CRISPR, in DNA differentiate, the number looks smoother
  * suitable for deep learning
- GPU for google compute engine
  * traning and inference: TensorFlow
  * Rendering for visual effects: V-Ray by ChaosGroup
  * Physical simulation and analysis (CFD, FEM, structural mechanism)
  * Real-time visual analytics and SQL db: MapD
  * FFT-based 3D protein docking: MEGADOCK
  * Faster than real-time 4k video transcoding: Colorfront Transkoder
  * Open source video transcoding: FFmepg, libav
  * Open source sequence mapping/alignment: BarraCUDA
  * subsurface analysis for the oil, gas: Reverse time migration
  * Risk management and ederivatives pricing: Computational Finance
  - Bare metal performance
  - flexible CPU count
- no more hardware administration
  * build-up yourself takes days
  * with gcloud, within hours
  * reduce cost
  * better performance (indexing went from daily to hourly)
  * flexibility
- Shazom
  * hear a segment of songs or TV shows find out in its database
- Schlumberger
  * an oil gas company
  * horizontal using google cloud
  * PBs of data per year
  * TBs of data per day
  * ground models usually costs 100TB or more
- Rendering on a GPU farm in the Cloud
  * render pipeline requirements
  * render 8k quality films in seconds with 70 GPUs with full ray-tracing
  * Nvidia Tesla P100 is going to release, currently K10
