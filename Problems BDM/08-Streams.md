# 08-Streams

## 8.1 **Problems**

1. Let’s suppose an arrival rate *λ* = 50*messages/sec* and a service rate *µ* = 50*messages/sec*. Given the message arrivals in the table below and a size of 100*bytes* per message, which is the minimum buffer size needed to guarantee that no message is lost?
    
    
    | Time | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | #Messages | 2 | 10 | 100 | 50 | 20 | 75 | 150 | 100 | 40 | 10 |
2. Let’s suppose a stationary situation in the processing of a stream, where the whole memory available is occupied by messages. Assume also the existence of a separated buffer big enough to allow us the complete processing of these messages already in memory without being concerned with new arrivals. Assume we have *M* = 10 memory pages with $R_s = 10$ messages per page. If the processing of these messages consists only of a lookup and in-memory processing/comparison of a message and a tuple is thousand times smaller than a disk access, give the cost in these two situations considering that every disk block of the table contains also *RT* = 10 tuples:
    1. Going through an index for each message with cost *h* = 3 for the index and one more disk access to the lookup table per message.
    2. Bringing each block of the lookup table (whose size is *B* = 100) into memory one by one and checking all messages in memory against all its tuples.
3. Let’s suppose that setting the execution environment for the processing of messages in a stream is *S* = 100 (e.g., placing & retrieving all the information to/from the stack), independently of their number. Both packing and unpacking the messages in a batch (i.e., a list of elements) have the same cost of $P_k = 1$ per message, and processing each message is $P_s = 10$.
    1. Which is the cost of truly streaming 10 messages one at a time?
    2. Which is the cost of processing one packed micro-batch of 10 messages at once?
4. Let’s suppose we have a log file recording the events coming from different machines. Thus, for each event we have the following information:
    
    $(logID, traceID, eventID, duration)$
    
    The *logID* corresponds to the IP of the machine; the *traceID* identifies the transaction inside the machine (i.e., two traceIDs can coincide in different machines); the *eventID* identifies the kind of action performed by the machine; finally, the *duration* is the number of milliseconds taken to implement the action. Assuming that we cannot keep all log entries in memory, and we decide to randomly sample them, **give the attributes** (up to three) you would use as parameters of the hash function implementing such sampling, so that each of the following queries gives a result as accurate as possible.
    
    1. Estimate the fraction of transactions where the same kind of action appears more than once.
        
        . . .
        
        . . .
        
        . . .
        
    2. For each machine, estimate the average number of actions per transaction.
    . . .
        
        . . .
        
        . . .
        
    3. Estimate the number of transactions with more than two actions taking more than 100ms.
    . . .
        
        . . .
        
        . . .
        
    4. Estimate the average sum of the duration per transaction.
        
        . . .
        
        . . .
        
        . . .
        
5. Let’s suppose we have a black-list of IP addresses (whose packages we do not want to cross our firewall), which is too long to be kept in memory ($10^7$ elements). Thus, we decide to implement a **Bloom filter** (to avoid further processing of black-listed addresses), with only $10^8$ bits.
    1. How many hash functions would you use?
    2. What’s the probability of a false positive in that case?
    3. Briefly explain what is the consequence of a false positive in this concrete case.
6. Let’s suppose we have a log file recording the events coming from different machines. Thus, for each event we have the following information:
    
    $(logID, traceID, eventID, duration)$
    
    The *logID* corresponds to the IP of the machine; the *traceID* identifies the transaction inside the machine; the *eventID* identifies the kind of action performed by the machine; finally, the *duration* is the number of milliseconds taken to implement the action. Consider the following table and assume that each machine generates the same number of events and at the same pace, and use an **exponentially decaying window model** with a constant *c* = 0*.*5.
    
    | Time | logID | traceID | eventID | duration |
    | --- | --- | --- | --- | --- |
    | 1 | 1 | 1 | A | 1 |
    | 2 | 2 | 1 | B | 100 |
    | 3 | 1 | 1 | C | 10 |
    | 4 | 2 | 1 | D | 10 |
    | 5 | 1 | 1 | A | 100 |
    | 6 | 2 | 1 | C | 1 |
    1. Give the milliseconds (and details of calculation) used per machine (i.e., for Machine 1 and Machine 2) when the last event arrives, provided that the current milliseconds are more valuable than the oldest ones.
        
        <aside>
        💡 Note: ln 2 *≈* 0.693
        
        </aside>
        
7. Assume we ingest a stream with an event every time a ticket is sold at a theater. Precisely, the stream has the following structure:
    
    $(movieID, theaterID, timestamp, price)$
    
    Next, we ingest the following ordered set of events:
    
    - (*m*1*, t*1*,* 12*h,* 10e)
    - (*m*2*, t*1*,* 14*h,* 12e)
    - (*m*2*, t*2*,* 15*h,* 18e)
    - (*m*1*, t*3*,* 18*h,* 6e)
    - (*m*3*, t*2*,* 19 : 15*h,* 13e)
    - (*m*1*, t*3*,* 19 : 30*h,* 10e)
    - (*m*2*, t*3*,* 19 : 45*h,* 25e)
    - (*m*3*, t*1*,* 20*h,* 17e)
    - (*m*1*, t*3*,* 20 : 30*h,* 10e)
    - (*m*2*, t*2*,* 21*h,* 8e)
    
    Provide a detailed answer (i.e., describe the process) for the following questions:
    
    1. Which theaters would be considered heavy hitters by the algorithm, given a required frequency of 50%?
    2. Under the exponentially-decaying window model, using an aging constant of *c* = 0*.*1 and a purging threshold of 0*.*6 (i.e., we’ll remove movies as soon as their popularity falls strictly bellow 0*.*6). What would be the most popular movies at 21h, considering an aging tick every 15 minutes? Give the value corresponding for each of the movies in that moment.
8. If you have a stream of messages that arrive at a rate of 10,000 per second accounting for 512*MB* per hour, how many RDDs will process your Spark Streaming process in one hour? Explicit all the assumptions you make.
9. Assume we ingest a stream with an event every time a ticket is sold at a theater. Precisely, the stream has the structure (*movieID, theaterID, timestamp, price*). Next, we ingest the following ordered set of events:
    - Event 1: (*m*3*, t*4*,* 12*h,* 10$)
    - Event 2: (*m*1*, t*2*,* 13*h,* 17$)
    - Event 3: (*m*2*, t*4*,* 14*h,* 11$)
    - Event 4: (*m*4*, t*1*,* 15*h,* 8$)
    - Event 5: (*m*1*, t*3*,* 16*h,* 9$)
    - Event 6: (*m*3*, t*4*,* 17*h,* 5$)
    - Event 7: (*m*6*, t*1*,* 18*h,* 15$)
    - Event 8: (*m*5*, t*2*,* 19*h,* 12$)
    - Event 9: (*m*7*, t*5*,* 20*h,* 17$)
    - Event 10:(*m*1*, t*1*,* 21*h,* 11$)
    
    Which theaters would be considered heavy hitters (using the approximate method) considering a required frequency of 33%? Provide a detailed answer (i.e., describe the content of the auxiliary structure after processing every message).