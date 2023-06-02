# 02-Distributed Data

## 2.1 Theoretical questions

1. Which kind of database is this according to the distribution of data? 
    
    ![02-Distributed%20Data/image1.jpeg](02-Distributed%20Data/image1.jpeg)
    
    
    **Centralized**
    
2. Explain what is (a) a distributed system, and (b) a parallel system. Compare both of them (i.e., what has one and not the other and vice-versa).

**distributed system contain multiple nodes (communicate through a network), parallel can be in one machine**

4. Which two kinds of schema information contains the Global Conceptual Schema that does not contain the Local Conceptual Schema in the Extended ANSI-SPARC Architecture for DDBMS?
    1. **Fragmentation**
    2. **Allocation**
5. In the context of distributed data management, name the **four big challenges** that need to be carefully considered in the presence of distribution from the tenant/user point of view.
    1. **Data Design**
    2. **Catalogue Management**
    3. **Transaction Management**
    4. **Query Processing**
6. Name the three characteristics of fragmentation that make it correct.
    1. **Complete**
    2. **Disjoint**
    3. **Reconstruct**
7. Which is the main problem in having replicas, and which is the innovation introduced by some NOSQL tools to solve it.
    - Problem:
    - Innovation:
8. Given *N* replicas, let’s call *R* the *ReadConcern* parameter of MongoDB and *W* the *WriteConcern* (which indicate respectively the number of copies it reads and writes, before confirming the operation to the user); give the equation involving those variables that corresponds to the **eventually consistent** configuration.
9. What is the difference between query cost and query response time ...
    1. in centralized systems?
    2. in distributed systems?
10. Name the two factors that make impossible having linear scalability according to the Universal Scalability Law.
    1. —
    2. —

## 2.2 Problems

1. Briefly explain (a) which fragmentation strategy has been applied for the tables below and whether this fragmentation strategy is (b) complete, (c) disjoint and (d) allows to reconstruct the global relations (if so, (e) indicate the operation).
    
    **Global Relations**
    
    ```jsx
    Kids(kidId, name, address, age)
    Toys(toyId, name, price)
    Requests(kidId, toyId, willingness)
    
    Note that requests(kidId) is a foreign key to kids(kidId) and
    similarly, requests(toyId) refers to toys(toyId).
    ```
    
    **Fragments**
    
    ![Untitled](02-Distributed%20Data/Untitled.png)
    
2.  You are a customer using an e-commerce based on heavy replication (e.g., Amazon):
    1. Show a database replication strategy (e.g., sketch it) where:
        1. You buy an item, but this item does not appear in your basket.
        2. You reload the page: the item appears.
        
        ⇒ What happened?
        
    2. Show a database replication strategy (e.g., sketch it) where:
        1. You delete an item from your command, and add another one: the basket shows both items.
        
        ⇒ What happened? Will the situation change if you reload the page?
        
3. Consider the following architectures and answer the questions:
    
    ![Figure 1: Centralized architecture](02-Distributed%20Data/image2.png)
    
    Figure 1: Centralized architecture
    
    | Type | Latency | Bandwidth |
    | --- | --- | --- |
    | Disk | ≈ 5 × 10^−3s (5 millisec.) | At best 100 MB/s |
    1. How long would it take (i.e., response time) to read 1TB with sequential access (Figure 1)? (in secs)
    2. How long would a single random access (i.e., reading one tuple, of for example 100 bytes, through an index) take (i.e., response time), assuming we already have the physical address? (in secs)
        
        ![Figure 2: Shared-memory architecture](02-Distributed%20Data/image3.png)
        
        Figure 2: Shared-memory architecture
        
        | Type | Latency | Bandwidth |
        | --- | --- | --- |
        | Disk | ≈ 5 × 10^−3s (5 millisec.) | At best 100 MB/s |
    3. How long would it take (i.e., response time) to read 1TB with parallel access (Figure 2)? Assume 100 disks (i.e., 100 replicas of the whole data) on the same machine with shared-memory and infinite CPU capacity.
    4. How long would a single random access (i.e., reading one tuple, of 100 bytes, through an index) take (i.e., response time), assuming we already have the physical address? (in secs)
        
        ![Figure 3: Shared-nothing architecture](02-Distributed%20Data/image4.png)
        
        Figure 3: Shared-nothing architecture
        
        | Type | Latency | Bandwidth |
        | --- | --- | --- |
        | Disk
        LAN
        Internet | ≈ 5 × 10^−3s (5 millisec.)
        ≈ 1 − 2 × 10^−3s (1-2 millisec.)
        Highly variable
        (typically 10-100ms) | At best 100 MB/s
        ≈ 1 GB/s (single rack) ≈ 10MB/s (switched)
        Highly variable
        (typically a few MB/s, e.g., 10MB/s) |
        
        > Note 1: It is approx. one order of magnitude faster to exchange main memory data between 2 machines in a data center, that to read on the disk.
        > 
        > 
        > *Note 2:* Exchanging through the Internet is slow and unreliable with respect to LANs.
        > 
    5. How long would it take (i.e., response time) to read 1TB with distributed access (Figure 3)? Assume 100 shared-nothing machines (with all data replicated in each of them) in a star-shape LAN in a single rack where all data is sent to the center of the star in only one network hop.
    6. How long would it take (i.e., response time) to read 1TB with distributed access (Figure 3)? Assume 100 shared-nothing machines (with all data replicated in each of them) in a star-shape cluster of ma- chines connected through the Internet where all data is sent to the center of the star in only one network hop.
    7. How long would a single random access (i.e., reading one tuple, of 100 bytes, through an index) take (i.e., response time), assuming we already have the physical address? (in secs)
4. What are the main differences between these two distributed access plans? Under which assumptions is one or the other better?
    
    ```sql
    SELECT * FROM employee e, assignedTo a
    WHERE e.#emp = a.#emp AND a.responsability = 'manager'
    
    ---
    
    Employee(#emp, empName, degree)
    S4: E1 = Employee(#emp ≤ ’E3’)
    S3: E2 = Employee(#emp > ’E3’)
    AssignedTo(#emp, #proj, responsability, fullTime)
    FK: #emp references Employee
    S1: A1 = AssignedTo(#emp ≤ ’E3’)
    S2: A2 = AssignedTo(#emp > ’E3’)
    ```
    
    ![02-Distributed%20Data/image5.png](02-Distributed%20Data/image5.png)
    
    ![02-Distributed%20Data/image6.png](02-Distributed%20Data/image6.png)
    
    Figure 4: Distributed Access Plans
    
5. Compute the fragment query (data location stage) for the database setting and query below and find in how many ways we can assign the operations to the different sites.
    
    ⇒ Database setting:
    
    - A distributed database with 5 sites (i.e., database nodes): *S*1, *S*2, *S*3, *S*4 and *S*5.
    - 3 relations in the database R, S and T
    - Each relation is horizontally fragmented in two fragments (we refer to them by the name of the relation and a subindex, for example: *R*1, *R*2). You can consider them to be correct (i.e., complete, disjoint and reconstructible).
    - Each fragment is replicated at all sites.
    - We have the following query:
        
        ![Untitled](02-Distributed%20Data/Untitled%201.png)
        
    
    ⇒ Process tree of the query:
    
    ![02-Distributed%20Data/image7.jpeg](02-Distributed%20Data/image7.jpeg)
    
6. Consider a left-deep process tree corresponding to a query, where each internal node is a join, and every leaf a data source (e.g., relational table). Knowing that the tree contains 9 nodes (including leaves), the system has as much parallelism capacity as needed to run all the joins in pipelining mode (no other kind of parallelism is available), which is the occupancy if the overall cost of the serial query is 4 seconds? Explicit any assumption you need to make.
