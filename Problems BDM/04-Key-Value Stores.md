# 04-Key-Value Stores

## 4.1 Theoretical questions

1. Which is the main difference between the hash functions used in the linear hash and consistent hash algorithms?

Linear: 2 hash functions
Consistent: A single hash function

2. With respect to distributed systems, explain what is a distributed hash table (DHT), and provide a brief description of how consistent hashing guarantees balancing keys when adding new servers.

Need to rehash the keys from the adjoining servers, but need not hash everything

## 4.2 Problems

1. Let’s assume we have a *Consistent hash* with *D* = 16, and the hash function is simply the module of the IP address or the key, and suppose the current state of the consistent hash is (position in the $ring:key|key|...$):
    
    ![Untitled](04-Key-Value%20Stores/Untitled.png)
    
    1. What happens when we insert objects 30 and 58? Draw the result.
    2. What happens in the structure when we register a new server with IP address 37? Draw the result.

Hash function: IP/key % D, i.e., modulo 16
Object 30: 30 % 16 = 14. Thus, 14: 46 | 30
Object 58: 58 % 16 = 10. If the modulo is not present in the ring, we add it to its successor. Since, 10 is not there, we add it to 12. Thus, 12: 12 | 43 | 58
IP with address 37: 37 % 16 = 5. Thus, 5: 37. Also, we need to rebalance keys from 4 and 7 to 5. From 4, keys satisfying x % 16 are None. From 7, keys satisfying x % 16 are 21 and 5.
Thus, 4: 36 | 20 | 4 | 3 | 19 | 18, while 5: 21 | 5 | 37 and 7: 38 | 23.

2. Let’s suppose we have a *Linear Hash* and the hash function is simply the module of the key, the capacity of a bucket is only four entries, and current state of the linear hash is $(bucketID: key|key|...)$:
    
    ![Untitled](04-Key-Value%20Stores/Untitled%201.png)
    
    a) What happens in the structure when we insert keys 14, 27, 37, and 44? Draw the result.

2n <= servers < 2n+1

Servers = 6, thus 22 <= 6 < 23; implying n = 2. Thus, we have 2 hash functions: key % 4 and key % 8.

Insert 14: 14 % 4 = 2, 14 % 8 = 6 (doesn’t exist), thus choosing 14 % 4 = 2. Thus, 2: 18 | 38 | 46 | 14

Insert 27: 27 % 4 = 3. Thus, 3: 43 | 19 | 3 | 15 -> 23 | 27

Insert 37: 37 % 4 = 1. Since, 1 is above pointer, we choose the hash function of 37 % 8 = 5. Thus 5: 21 | 5 | 37

Insert 44: 44 % 4 = 0. Since, 0 is above pointer, we choose the hash function of 44 % 8 = 4, now there is an overflow. Thus, the bucket with the pointer splits, i.e., 2. Pointer moves to p = 3 and 4:  4 | 36 | 20 | 12 -> 44.

Moving keys from bucket 2: 18 % 8 = 2, 38 % 8 = 6, 46 % 8 = 6 and 14 % 8 = 6. Thus, 6: 38 | 46 | 14 and 2: 18.

    
3. Let’s suppose that, we have an *LSM Tree* that reached the threshold to consider the MemStore is full, and it contains four entries with format [*key, value, timestamp*] needing ten characters each. The content of the different structures is:
    - MemStore: [1*, v, t*50]*,* [15*, v, t*49]*,* [17*, v, t*47]*,* [29*, v, t*48]
    - Commit Log: [17*, v, t*47]*,* [29*, v, t*48]*,* [15*, v, t*49]*,* [1*, v, t*50]
    - SSTable: [13*, v, t*23]*,* [25*, v, t*17]*,* [35*, v, t*40]*,* [59*, v, t*38]
    - Index: [13*,* 0]*,* [25*,* 30]*,* [35*,* 60]*,* [59*,* 90]
    
    Assuming that the minimum size of an SSTable is 120 characters and on having two SSTables a minor compactation is automatically triggered, explicit the content of all structures once the compactation is done.
    
    - MemStore: NULL
    - Commit Log: NULL
    - SSTable: [1, v, t50], [13, v, t23], [15, v, t49], [17, v, t47], [25, v, t17], [29, v, t48], [35, v, t40], [59, v, t38] 
    - Index: [1, 0], [13, 30], [15, 60], [17, 90], [25, 120], [29, 150], [35, 180], [59, 210]

4. Briefly explain what is wrong in this linear hash structure, or if you think it is right, explicitly say so. 
    
    ![Untitled](04-Key-Value%20Stores/Untitled%202.png)

Wrong, bucket 2 should follow hash of %2^(n+1)
    
5. Suppose you have a hash function whose range has size 100 (i.e., D=100), and a Consistent Hash structure with 5 machines (M1...5) whose identifiers map to values *h*(*M*1) = 0, *h*(*M*2) = 20, *h*(*M*3) = 40, *h*(*M*4) = 60, *h*(*M*5) = 80. What happens if you have an object mapped to value *h*(*O*) = 90?

Goes to machine h(M1) = 0

6. Given an empty **Consistent Hash** with *h*(*x*) = *x*%32 (i.e., we directly take module 32 to both the keys and the bucket IDs), and unlimited capacity in each bucket, consider you have a cluster of four machines with IDs 19, 22, 75, 92, and draw the result of inserting the following keys in the given order: 12, 4, 10, 49, 42, 60, 63, 53, 47, 27, 26, 28, 13, 52.

Machine ID: 75 => [4, 10, 42, 63]
Machine ID: 19 => [12, 49, 47, 13]
Machine ID: 22 => [53, 52]
Machine ID: 92 => [60, 27, 26, 28]

7. Suppose you implement a system to store images in hundreds of machines with thousands of users using HBase with a single column-family. These images taken at time VT belong to a person P who tags each with a single subject S (e.g., family, friends, etc.) and are concurrently uploaded into the system at time TT in personal batches containing multiple pictures of different subjects taken at different times. Each person can then retrieve all his/her pictures of one single subject that were taken after a given time. Precisely define the key you would use if you exclusively prioritize (i.e., do not consider any other criteria)...
    - Load balancing on ingestion:
        
        ⇒ Assumptions made:
        
    - Load balancing on querying:
        
        ⇒ Assumptions made:
        
    - I/O cost (i.e., minimum blocks retrieved) on ingestion:
        
        ⇒ Assumptions made:
        
    - I/O cost (i.e., minimum blocks retrieved) on querying:
        
        ⇒ Assumptions made:
        
8. Given an empty **linear hash** with *f* (*x*) = *x* (i.e., we directly apply the module to the keys), and a capacity of four keys per bucket, draw the result of inserting the following keys in the given order: 12, 4, 10, 49, 42, 60, 63, 53, 47, 27, 26, 28, 13, 52.