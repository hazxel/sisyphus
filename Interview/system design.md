
# Alex Xu

### Framework for system design interviews

1. Understand the problem and establish design scope
2. Propose high-level design and reach an agreement with interviewer
3. Design deep dive, identify and prioritize components in the architecture.
4. Warp up, the interviewer might ask you a few follow-up questions or give you the freedom to discuss other additional points. 

### RDBMS vs NoSQL

Relational databases represent and store data in tables and rows. You can perform join
operations using SQL across different database tables.

Non-relational databases are grouped into four categories, and join operations are generally not supported in non-relational databases: 

- key-value stores
- graph stores
- column stores
- document stores: used to store a massive amount of data.

Non-relational databases might be the right choice if:

- Your application requires super-low latency.
- Your data are **unstructured**, or you do not have any relational data.
- You only need to serialize and deserialize data (JSON, XML, YAML, etc.).
- You need to store a massive amount of data.
- 无固定pattern，不保证ACID，容易横向扩展

### Vertical scaling vs horizontal scaling


Vertical scaling, referred to as “scale up”, means the process of adding more power (CPU, RAM, etc.) to your servers. Horizontal scaling, referred to as “scale-out”, allows you to scale by adding more servers into your pool of resources.


When traffic is low, vertical scaling is a great option, and the simplicity of vertical scaling is its main advantage. Unfortunately, it comes with serious limitations.

- Vertical scaling has a hard limit. It is impossible to add unlimited CPU and memory to asingle server.

- Vertical scaling does not have failover and redundancy. If one server goes down, thewebsite/app goes down with it completely.

  Horizontal scaling is more desirable for large scale applications due to the limitations of vertical scaling.

### Load balancer

A load balancer evenly distributes incoming traffic among web servers. If the website traffic grows rapidly, the load balancer can add more servers to the web server pool, and the load balancer automatically starts to send requests to them.

Users connect to the public IP of the load balancer directly. For better security, private IPs are used for communication between servers. A private IP is an IP address reachable only between servers in the same network and is unreachable over the internet.

### Database replication

Database replication can be used in many database management systems, usually with a master/slave relationship between the original (master) and the copies(slaves).

A master database generally only supports write operations. A slave database gets copies of the data from the master database and only supports read operations. All the data-modifying commands like insert, delete, or update must be sent to the master database. Most applications require a much higher ratio of reads to writes; thus, the number of slave databases in a system is usually larger than the number of master databases. Advantages:

- Better performance: This model offers higher throughput because it allows more queries to be processed in parallel.

  - Reliability: You do not need to worry about data loss because data is replicated across multiple locations.

- High availability: Even if a database is offline as you can access data stored in another database
  server.


  ​			