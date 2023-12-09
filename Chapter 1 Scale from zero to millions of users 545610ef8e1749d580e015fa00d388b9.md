# Chapter 1: Scale from zero to millions of users

## Journey of building a complex system:

- Single server setup: Everything is running on a single server → Web, app, server, database, cache.
- Separate server for web/mobile traffic and one for database. This separation allows them to be scaled independently.
- Separate servers for dealing with app/web traffic with a load balancer exposed using a public ip address.
- Master slave databases for handling writes and reads respecitvely with replication across geographical locations for fault tolerance and disaster resistance.

## When to use a NoSql DB:

- Your application requires super-low latency
- Your data is unstructured or you do not have any relational data
- You only need to serialize and deserialize data (JSON, XML,YAML etc). No need for joins etc
- You need to store a massive amount of data

## Vertical vs Horizontal scaling:

- Vertical scaling referred to as Scale up means the process of adding more computational power (Ram, Gpu, cpu) to your servers. Vertical scaling does not have any failover or redundancy and there is always a limit to how much power you can add to your server.
- Horizontal scaling referred to as Scale Out allows you to scale by adding more servers into your pool of resources.

## Need for a Load balancer:

- Without a load balancer, the users are connected to the web/app server directly. Users will be unable to access the web/app server directly if the server goes down. In another scenario, if many users access the same server, then response time will go up.
- A load balancer would evenly distribute traffic among web servers that are defined in a load balanced set. The load balancer’s public ip will be exposed to the users and the load balancer will communicate with the servers using private ip. Similarly inter server or server-db communication would be done using private ip’s.

## **Database replication:**

- Database replication is usually implemented in database management systems using master/ slave relationship. Master database supports write operations. The slave database gets copies of the data from the master db itself and only supports read operations.
- Most applications require much higher ratio of reads to writes and hence have a higher ratio of slaves to masters.

## Advantages of database replication:

- Read/ write operation distribution
- Replication if data across multiple geographical locations ensures **reliability** in case the database server is destroyed by a natural disaster like an earthquake. This also ensures **availability** as your website remains in operation as you can access the data stored in another database server.

## Handling scenario where database goes offline:

- Slave database going offline → Read operations will be redirected to the master database. As soon as the issue is found, another slave will replace the older one. In case multiple slaves are available, read operations will be redirected to the other healthy slaves.
- Master database goes offline → Slave database will be promoted to be the new master. All database operations will be performed on the new master database. A new slave database will replace the old one for data replication immediately. 
In production databases, promoting a new master is more complicated as the data in the slave database might not be upto date. The missing data will need to be updated using some database replication scripts.
(There are some other complicated replication methods like multi-masters and circular replication)

### System design after adding load balancer and database replication:

![Screenshot 2023-02-19 at 1.13.32 PM.png](Chapter%201%20Scale%20from%20zero%20to%20millions%20of%20users%20545610ef8e1749d580e015fa00d388b9/Screenshot_2023-02-19_at_1.13.32_PM.png)

## Cache:

- Cache is a temporary ( Because it is stored in RAM) storage area that stores the result of expensive responses or frequently accessed data in memory so that subsequent requests are served more quickly. Otherwise each time a new web page loads, we need to make a DB hit. This will impact app performance badly.
- Cache tier is a temporary data store layer which is much faster than the database → Having a spearate cache tier enables us to scale this independently too.

![Screenshot 2023-02-19 at 1.17.38 PM.png](Chapter%201%20Scale%20from%20zero%20to%20millions%20of%20users%20545610ef8e1749d580e015fa00d388b9/Screenshot_2023-02-19_at_1.17.38_PM.png)

^ The above diagram represents a read-through cache.

## Considerations for using a cache:

- **Decision to use a cache:** Use cache when data is read frequently but modified infrequently. Since cached data is stored in volatile memory, a cache server is not the right place to persist data. If a cache server restarts, all the data in memory is lost. Thus important data should be stored inside a database itself.
- **Expiration policy:** It’s a good practice to implement an expiration policy. Once cached data is expired, it is removed from the cache. Expiration data should not be too long since it will lead to data becoming too stale. Similary making it too short will lead to frequent db hits.
- **Consistency of data store and cache:** This involves keeping the data store and cache in sync. This inconsistency can happen because the operations to modify cache and data store do not happen in a single transaction. When scaling across multiple regions, maintaining this consitency is challenging. Multiple cache servers across different data centres are recommeded to avoid a single point of failure.

[Scaling Memcache at Facebook - Meta Research | Meta Research](https://research.facebook.com/publications/scaling-memcache-at-facebook/)

[SREcon15 - Learning from Mistakes and Outages at Facebook](https://www.youtube.com/watch?v=J_vcEaU21eI&t=9s)

- **Eviction policy:** Once cache is full, any requests to add items to the cache might cause existing items to be removed. This is called cache eviction. Least recently used (LRU) is the most popular cache eviction policy. There are others such as Least frequently used (LFU) and First in first out (FIFO).