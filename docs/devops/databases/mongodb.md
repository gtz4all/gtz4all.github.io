---
title: "MongoDB"
description: "MongoDB is a NoSQL Database"
---

Over the past years, I've been searching for an easy to use database that can interact with other tools like ansible and python. Since I didnt have one at the moment, I started using CVS files with the plan to migrate to a database in the future. MongoDB is an open-source, document-oriented, NoSQL database program which uses JSON-like document schemas, making it easy for other tools to work with it. MongoDB provides tools that can import/export CSV files. MongoDB being an non-rational database can be used to store different type of data such as key-value pairs, JSON data, CLI configuration and even images
https://www.mongodb.com

**Document model Database**

  Document-Oriented NoSQL DB stores and retrieves data as a Key-value pair, but the value part is stored as a document in JSON or XML formats. 

**Some MongoDB Advatanges**

  1. Store unstructured, semi-structured, or structured data.
  2. Uses easy to update,flexible schemas and fields.
  3. Open Source 

**Mongo DB Structure**

Type|description | usage
----|------------|------
Databases|hold one or more collections of documents|no create option instead you just try to use it and it gets created.
Collection|stores documents, analogous to rational database tables|same as database, applies to MongoDB CLI and Python.

* Mongo-Express

  Mongo-Express is an open-source Web-based MongoDB admin interface written with Node.js, Express and Bootstrap3. It provides an easy to read, visual represtation of our databases
  https://github.com/mongo-express/mongo-express

## MongoDB Test Drive
---
Using docker to test drive both mongodb and mongo-express

* Mongo Dockerfile

  This dockerfile creates both a mongodb and mongo-express instance

```yaml
version: '3.1'

services:
  mongodb:
    container_name: mongodb
    image: mongo
    restart: always
    environment:
      ## MongoDB Credentials
      MONGO_INITDB_ROOT_USERNAME: ${DBUSER}
      MONGO_INITDB_ROOT_PASSWORD: ${DBPASS}

  mongo-express:
    container_name: mongogui
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ## MongoDB Container Name
      ME_CONFIG_MONGODB_SERVER: mongodb
      ## MongoDB Credentials
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${DBUSER}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${DBPASS}
      ## MongoDB Web UI Credentials
      ME_CONFIG_BASICAUTH_USERNAME: ${DBUSER}
      ME_CONFIG_BASICAUTH_PASSWORD: ${DBPASS}
```

* Run Containers
    ```sh
    DBUSER=admin DBPASS=admin docker-compose up -d
    ```

## MongoDB CLI 

* Install
    ```sh
    sudo apt update && sudo apt install mongodb-clients -y
    ```

* Export Credentials

    ```sh
    export DBUSER=admin DBPASS=admin
    echo $DBUSER
    echo $DBPASS
    ```

* Mongo Container IP

    ```sh
    export MONGODB_IP=$(docker inspect mongodb --format '{range.NetworkSettings.Networks }{.IPAddress}{ end }')
    echo $MONGODB_IP
    ```

* Connect to MongoDB

    ```sh
    mongo mongodb://$MONGODB_IP:27017/ -u $DBUSER -p $DBPASS --authenticationDatabase "admin"
    ```

* MongoDB CLI connection output

    ```sh
    :~$ mongo mongodb://$MONGODB_IP:27017/ -u $DBUSER -p $DBPASS --authenticationDatabase "admin"
    MongoDB shell version v3.6.3
    connecting to: mongodb://172.18.0.2:27017/
    MongoDB server version: 4.4.6
    WARNING: shell and server versions do not match
    Server has startup warnings:
    {"t":{"$date":"2021-06-07T18:10:19.421+00:00"},"s":"I",  "c":"STORAGE",  "id":22297,   "ctx":"initandlisten","msg":"Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem","tags":["startupWarnings"]}
    {"t":{"$date":"2021-06-07T18:10:20.787+00:00"},"s":"W",  "c":"CONTROL",  "id":22167,   "ctx":"initandlisten","msg":"You are running on a NUMA machine. We suggest launching mongod like this to avoid performance problems: numactl --interleave=all mongod [other options]","tags":["startupWarnings"]}
    > 
    ```

* List all databases

    ```sh
    > db.adminCommand( { listDatabases: 1 } )
    {
            "databases" : [
                    {
                            "name" : "admin",
                            "sizeOnDisk" : 102400,
                            "empty" : false
                    },
                    {
                            "name" : "config",
                            "sizeOnDisk" : 12288,
                            "empty" : false
                    },
                    {
                            "name" : "local",
                            "sizeOnDisk" : 73728,
                            "empty" : false
                    }
            ],
            "totalSize" : 188416,
            "ok" : 1
    }
    >
    ```

* Create new database

    ```sh
    use DataCenterDB
    ```

* create a collection and insert items 

    ```sh
    ## use - creates database
    use DataCenterDB
    ```
    ```sh
    ## db.[collection-name].insert - creates collection if it does not already exist
    db.DataCenter1.insert([
      {
        "name": "COREROUTER1",
        "tag": "CORE",
        "asn": "64521",
        "lo0": "10.168.1.1"
      },
      {
        "name": "COREROUTER1",
        "tag": "CORE",
        "asn": "64522",
        "lo0": "10.168.1.2"
      },
      {
        "name": "DISTROUTER1",
        "tag": "DIST",
        "asn": "64523",
        "lo0": "10.168.1.3"
      },
      {
        "name": "DISTROUTER1",
        "tag": "DIST",
        "asn": "64521",
        "lo0": "10.168.1.2"
      }
    ])
    ```

* Insert output

    ```sh
    BulkWriteResult({
            "writeErrors" : [ ],
            "writeConcernErrors" : [ ],
            "nInserted" : 4,
            "nUpserted" : 0,
            "nMatched" : 0,
            "nModified" : 0,
            "nRemoved" : 0,
            "upserted" : [ ]
    })
    ```

* Query database

    **find()**: query data from MongoDB collection.
    
    **pretty()**: display the results in a JSON formatted way.

    ```sh
    ## db.[collection-name].find()
    use DataCenterDB
    > db.DataCenter1.find()
    { "_id" : ObjectId("60be987af6652ebfdef9b958"), "name" : "COREROUTER1", "tag" : "CORE", "asn" : "64521", "lo0" : "10.168.1.1" }
    { "_id" : ObjectId("60be987af6652ebfdef9b959"), "name" : "COREROUTER1", "tag" : "CORE", "asn" : "64522", "lo0" : "10.168.1.2" }
    { "_id" : ObjectId("60be987af6652ebfdef9b95a"), "name" : "DISTROUTER1", "tag" : "DIST", "asn" : "64523", "lo0" : "10.168.1.3" }
    { "_id" : ObjectId("60be987af6652ebfdef9b95b"), "name" : "DISTROUTER1", "tag" : "DIST", "asn" : "64521", "lo0" : "10.168.1.2" }
    > db.DataCenter1.find().pretty()
    {
            "_id" : ObjectId("60be987af6652ebfdef9b958"),
            "name" : "COREROUTER1",
            "tag" : "CORE",
            "asn" : "64521",
            "lo0" : "10.168.1.1"
    }
    {
            "_id" : ObjectId("60be987af6652ebfdef9b959"),
            "name" : "COREROUTER1",
            "tag" : "CORE",
            "asn" : "64522",
            "lo0" : "10.168.1.2"
    }
    {
            "_id" : ObjectId("60be987af6652ebfdef9b95a"),
            "name" : "DISTROUTER1",
            "tag" : "DIST",
            "asn" : "64523",
            "lo0" : "10.168.1.3"
    }
    {
            "_id" : ObjectId("60be987af6652ebfdef9b95b"),
            "name" : "DISTROUTER1",
            "tag" : "DIST",
            "asn" : "64521",
            "lo0" : "10.168.1.2"
    }
    ## Find an specific item
    > db.DataCenter1.find({"tag": "DIST"}).pretty()
    {
            "_id" : ObjectId("60be987af6652ebfdef9b95a"),
            "name" : "DISTROUTER1",
            "tag" : "DIST",
            "asn" : "64523",
            "lo0" : "10.168.1.3"
    }
    {
            "_id" : ObjectId("60be987af6652ebfdef9b95b"),
            "name" : "DISTROUTER1",
            "tag" : "DIST",
            "asn" : "64521",
            "lo0" : "10.168.1.2"
    }
    >
    ```

## MongoDB Python 3 Interactive Mode 

* Install

    ```sh
    pip3 install pymongo
    ```

    * Interactive Mode 
    ```python
    import os
    import pymongo

    dbpass = os.environ.get('DBPASS')
    dbuser = os.environ.get('DBUSER')
    dbip = os.environ.get('MONGODB_IP')

    dbclient = pymongo.MongoClient (f"mongodb://{dbip}:27017/", username=dbuser, password=dbpass)
    ```

* list database option 1

    ```py
    for db in dbclient.list_databases():
        print(db)
    ```    
* list database option 2

    ```py
    print(list(dbclient.list_databases()))
    ```

* create database "DataCenterDB"

    ```py
    DCdb = dbclient["DataCenterDB"]
    ```

* create "DataCenter1" collection inside "DataCenterDB" 

    ```py
    DC1 = DCdb["DataCenter1"]

    ## Add items to Collection
    updatedb = DC1.insert([
      {
        "name": "COREROUTER1",
        "tag": "CORE",
        "asn": "64511",
        "lo0": "10.168.1.1"
      },
      {
        "name": "COREROUTER1",
        "tag": "CORE",
        "asn": "64512",
        "lo0": "10.168.1.2"
      },
      {
        "name": "DISTROUTER1",
        "tag": "DIST",
        "asn": "64513",
        "lo0": "10.168.1.3"
      },
      {
        "name": "DISTROUTER1",
        "tag": "DIST",
        "asn": "64511",
        "lo0": "10.168.1.2"
      }
    ])
    ```

* create "DataCenter1" collection inside "DataCenterDB" 

    ```py
    DC2 = DCdb["DataCenter2"]

    ## Add items to Collection
    updatedb = DC2.insert([
      {
        "name": "COREROUTER2",
        "tag": "CORE",
        "asn": "64521",
        "lo0": "10.168.2.1"
      },
      {
        "name": "COREROUTER2",
        "tag": "CORE",
        "asn": "64522",
        "lo0": "10.168.2.2"
      },
      {
        "name": "DISTROUTER2",
        "tag": "DIST",
        "asn": "64523",
        "lo0": "10.168.2.3"
      },
      {
        "name": "DISTROUTER2",
        "tag": "DIST",
        "asn": "64521",
        "lo0": "10.168.2.2"
      }
    ])
    ```

* list collections 

    ```py
    print(DCdb.list_collection_names())

    ## list output
    ['DataCenter2', 'DataCenter1']
    ```

* query all items in collection

    ```py
    print(list(DC1.find()))

    ## output
    [
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda4"")",
          "name":"COREROUTER1",
          "tag":"CORE",
          "asn":"64511",
          "lo0":"10.168.1.1"
      },
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda5"")",
          "name":"COREROUTER1",
          "tag":"CORE",
          "asn":"64512",
          "lo0":"10.168.1.2"
      },
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda6"")",
          "name":"DISTROUTER1",
          "tag":"DIST",
          "asn":"64513",
          "lo0":"10.168.1.3"
      },
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda7"")",
          "name":"DISTROUTER1",
          "tag":"DIST",
          "asn":"64511",
          "lo0":"10.168.1.2"
      }
    ]
    ```

* query an item in collection

    ```python
    print(list(DC1.find({ "tag": "DIST" })))

    ## output
    [
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda6"")",
          "name":"DISTROUTER1",
          "tag":"DIST",
          "asn":"64513",
          "lo0":"10.168.1.3"
      },
      {
          "_id":"ObjectId(""60beaac074d25b62b989fda7"")",
          "name":"DISTROUTER1",
          "tag":"DIST",
          "asn":"64511",
          "lo0":"10.168.1.2"
      }
    ]
    ```

* Conclusion

This is the end of the mongoDB demo

