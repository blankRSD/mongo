## Set up Sharding using Docker Containers

> Mongo Router (mongos) is now in port 27017 - This is accessible outside the machine

> Config Server is now on port 27019 - Taken from mongo docs

> Shard1 and its replica will be available on ports 27020, 27021, 27022 - since they are deployed in the same system. If not they would have been each on 27018

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://172.31.65.218:27019
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "172.31.65.218:27019" },
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://172.31.65.218:27020
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "172.31.65.218:27020" },
      { _id : 1, host : "172.31.65.218:27021" },
      { _id : 2, host : "172.31.65.218:27022" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://172.31.65.218:27017
```
Add shard
```
mongos> sh.addShard("shard1rs/172.31.65.218:27020,172.31.65.218:27021,172.31.65.218:27022")
mongos> sh.status()
```
## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://172.31.65.218:50004
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "172.31.65.218:50004" },
      { _id : 1, host : "172.31.65.218:50005" },
      { _id : 2, host : "172.31.65.218:50006" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://172.31.65.218:60000
```
Add shard
```
mongos> sh.addShard("shard2rs/172.31.65.218:50004,172.31.65.218:50005,172.31.65.218:50006")
mongos> sh.status()
```
### To Shard the database
1. Connect to mongos
2. Enable sharding on db
```
mongos> sh.enableSharding("demo")
```
3. Shard the collection
```
sh.shardCollection("demo.users", {"username": "hashed"})
db.users.getShardDistribution()
```

