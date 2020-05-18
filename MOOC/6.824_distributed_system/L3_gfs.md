# Lecture 3 GFS

## Overview
###### Why are we reading this paper?
- distributed storage is a key abstraction
    - what should the interface/semantics look like?
    - how should it work internally?
- GFS paper touches on many themes of 6.824
    - parallel performance, fault tolerance, replication, consistency
- good systems paper -- details from apps all the way to network
- successful real-world design

###### Why is distributed storage hard?
- high performance -> shard data over many servers
- many servers -> constant faults
- fault tolerance -> replication
- replication -> potential inconsistencies
- better consistency -> low performance

###### What would we like for consistency?
- Ideal model: same behavior as a single server
  - server uses disk storage
  - server executes client operations one at a time (even if concurrent)
  - reads reflect previous writes
    - even if server crashes and restarts
  - thus:
  ```
    C1: Wx1
    C2: Wx2
    C3:     Rx?
    C4:         Rx?
  ```  
  answer: either 1 or 2, but both have to see the same value.
  - This is a "strong" consistency model.
  - But a single server has poor fault-tolerance.

###### Replication for fault-tolerance makes strong consistency tricky.
![](images/2020-05-12-12-56-44.png)
- a simple but broken replication scheme:
    - two replica servers, S1 and S2
    - clients send writes to both, in parallel
    - clients send reads to either
  - in our example, C1's and C2's write messages could arrive in different orders at the two replicas
    - if C3 reads S1, it might see x=1
    - if C4 reads S2, it might see x=2
  - or what if S1 receives a write, but the client crashes before sending the write to S2?
  - that's not strong consistency!
  - better consistency usually requires communication to ensure the replicas stay in sync -- can be slow!
  - lots of tradeoffs possible between performance and consistency we'll see one today

## GFS
##### requiement
- Big, fast
- Global
- Sharding
- Automatic recovery

##### limits
- Single data-center
- Internal use
- Big sequential access (not small,not random access)
- Thoughput instead of latency


##### What was new about this paper in 2003?
- Not the basic ideas of distribution, sharding
- fault-tolerance.
- Huge scale.
- Used in industry, real-world experience.
- Successful use of weak consistency.
- Successful use of single master.


##### Overall structure
- `clients` (library, RPC -- but not visible as a UNIX FS)
- `chunk servers`, 
  - each file split into independent 64 MB chunks
  - each chunk replicated on 3 every file's chunks are spread over the chunk servers
  - for parallel read/write (e.g. MapReduce), and to allow huge files
- `single master` (!), and master replicas
  - division of work: master deals w/ naming, chunkservers w/ data

##### Master state
- in RAM (for speed, must be smallish):
```
  file name -> array of chunk handles (nv)
  chunk handle -> version # (nv)
                    list of chunkservers (v)
                    primary (v)
                    lease time (v)
```                    
- on disk:
  -  log
  - checkpoint                    


- Why a log? and checkpoint?
- Why big chunks?

##### client C wants to read a file
  1. C sends filename and offset to master M (if not cached)
  2. M finds chunk handle for that offset
  3. M replies with list of chunkservers only those with latest version
  4. C caches handle + chunkserver list
  5. C sends request to nearest chunkserver chunk handle, offset
  6. chunk server reads from chunk file on disk, returns

How does the master know what chunkservers have a given chunk?

##### client wants to do a "record append"?
  paper's Figure 2
  1. C asks M about file's last chunk
  2. if M sees chunk has no primary (or lease expired):
     2a. if no chunkservers w/ latest version #, error
     2b. pick primary P and secondaries from those w/ latest version #
     2c. increment version #, write to log on disk
     2d. tell P and secondaries who they are, and new version #
     2e. replicas write new version # to disk
  3. M tells C the primary and secondaries
  4. C sends data to all (just temporary...), waits
  5. C tells P to append
  6. P checks that lease hasn't expired, and chunk has space
  7. P picks an offset (at end of chunk)
  8. P writes chunk file (a Linux file)
  9. P tells each secondary the offset, tells to append to chunk file
  10. P waits for all secondaries to reply, or timeout
      secondary can reply "error" e.g. out of disk space
  11. P tells C "ok" or "error"
  12. C retries from start if error

The lease design is to avoid a primay problem
- "Split brain" also caused by net partition
- master must wait the lease to expire before it designate a new primary