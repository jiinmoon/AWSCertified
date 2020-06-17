# Elasticache

- It is a web service that makes it easy to deploy, operate, and scale an
    in-memory cache in the cloud.
- The service improves the performance of web apps by allowing you to retrieve
    infromation from fast, managed, in-memory caches, instead of relying
    entirely on slower disk-based databases.

- ElastiCache can be used to improve latency and throughput for many read-heavy
    app workloads (i.e. SNS, gaming, media sharing...) or compute intensive
    workloads (i.e. recommendation engine).

- Caching improves app performance by storing critical pieces of data in memory
    for low-latency access (by storing frequently used data).
- Cached information may include the results of I/O-intensive database queries
    or the results of computationally-intensive calculations.

## Types of ElastiCache

- Memcached:
    - A widely adopted memory object caching system. ElastiCache is protocol
        compliant with Memcached, so popular tools that you use today with
        existing Memcached environments will work.

- Redis:
    - A popular open-source in-memory key-value store that supports data
        structures such as sorted sets and lists.
    - ElastiCache supports Master / Slave replication and Multi-AZ which can be
        used to achieve cross AZ redundancy.

- Both Memcached and Redis appear similar, they are very different in practice.
- Because of the replication and persistence features of Redis, ElastiCache
    manages Redis more as a relational db. Redis ElastiCache clusters are
    managed as stateful entities that include failover, similar to how RDS
    manages db failover.
- Because Memcached is designed as a pure caching solution without persistence,
    ElastiCache manages memcached nodes as a pool that can grow and shrink,
    similar to EC2 Auto Scaling Group. Individual nodes are expendable, and
    ElastiCache provides additional capabilities here such as automatic node
    replacement and Auto Discovery.

## Use Cases: Memcached

- Is object caching your primary goal? i.e. to offload your database?
- Looking for simplest solution?
- Planning on running large cache nodes, and require multithreaded performance
    with utilization of multiple cores?
- Want ability to scale your cache horizontally as you grow?

## Use Cases: Redis

- Looking for more advanced data types such as lists, hashes, sets?
- Require sorting and ranking datasets in memory? i.e. creating leaderboard?
- Is persistence of your key store important?
- Want to run in multiple AWS Availability Zones (Multi-AZ) with failover?

---

# Exam Tips

- Typically, you will be given a scenario where a particular database is under a
    lot of stree/load; you will be asked which service you should use to
    alleviate this.

- ElastiCache is a good choice if your database is particularly read-heavy and
    not prone to frequent changing.

- Redshift is a good answer if the reason your database is felling stress is
    because management keep running OLAP transactions on it.

- Know use cases for each type:
    - Use Memcached if
        - object caching is primary goal.
        - want to keep things simple.
        - scale your cache horizontally.
    - Use Redis if
        - have advanced data types (lists, sets, hashes).
        - doing the data sorting and ranking.
        - data persistence.
        - Multi-AZ.
        - Pub/Sub capabilities.

