# Lesson 10 Consistency in Distributed Data Stores

## 10.1 Introduction

- Practical consideration for consistency
- Districuted data (key-value) stores
  - Memcached at Facebook
- Different consistency models
  - Causal + from COPS: Don't settle for Eventual Consistency

## 10.2 Why is Consistency Important and Hard?

![](images/2021-04-05-12-14-59.png)

- Distributed state
- Replication for availability and fault tolerance
- Caching for performance
- Failures

### Too Hard? Change the Model?

![](images/2021-04-05-12-20-50.png)

## 10.3 Key-Value Store

Key-Value Store Data Model

- Put(set) and Get
- Scan, range query
- More complex data models and types of operations

## 10.4 Memcached

![](images/2021-04-05-13-25-37.png)

- Simple Key-value store **originally at Facebook**
- Nishtala et al, **Scaling Memcache at Facebook, NSDI' 13**
- Replaced with **Tao, ATC' 13**

### Memcached

![](images/2021-04-05-13-28-10.png)

## 10.5 Look-Aside Cache Design

#### Look-Aside Cache Design

- **Workload is**
  - very read intensive
  - large data capacity
  - "hot data", temporal locality
- **Original strore in SQL DBs**

#### Design

- => **simple in-memory KV object cache**
- Look-aside & demand-filled
  - ![](images/2021-04-05-13-33-42.png)
- Db updates => cache deletions(先删 cache，再删 db)
- Discard based on LRU
- MC: clean, read-only data
- Non-authoritative
- Look-aside design makes some optimizations possible
- ![](images/2021-04-05-13-37-47.png)

## 10.6 Mechanisms in Memcached

- **Problem with look-aside cache:**
  ![](images/2021-04-05-13-58-07.png)
- **Solution: Lease**
  - Issued on cache miss
  - Detect concurrent writes
    - => oredering of writes maintained
  - Other examples:
    - Thundering heards
    - Serving stale values

#### Client-Side Request Routing

- **Scale cache with more MC**
  - finite memory
- **More MC instances**
  - data sharded
- **Routing in clients**
  - mcrouter
- **Multiple clusters**
  - Scaling beyond a single cluster
  - separate failure domains
- **Multiple MC caches**
  - DB-drives invalidations in commit order
    ![](images/2021-04-05-15-42-31.png)

#### Geo Distributed Replication

- **Cannot use same consistency mechanisms** for multiple clusters
- **Combine with replication** at the storage layer
  ![](images/2021-04-05-15-46-47.png)
  marker 可以锁住之后的操作，保证一致性

## 10.7 Causal+ Consistency

![](images/2021-04-30-13-14-00.png)

- **Causal dependencies not visible**
  - Embedded in application semantics
  - (Geo)replication, caching, ..., all reads/writes are not visible in a single location
- => **COPS to the rescue**
  - Don's settle for eventual: Scalable Causal Consistency for Wide-Area Storage with COPS", - Wyatt Lloyd, et. all, SOSP'11

**GET**
![](images/2021-04-05-15-55-28.png)
**PUT**
![](images/2021-04-05-15-55-59.png)
**Causal+Replication**
![](images/2021-04-05-15-56-56.png)

## 10.8 Summary

- Consistency: models, tradeoffs, techniques
- Memcache architecture and design decisions
- Casual + Consistency with COPS
