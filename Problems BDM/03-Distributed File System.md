# 03-Distributed File System

## 3.1 Theoretical questions

1. How does HDFS decide where to place the different chunks of a file? 

## Problems

1. Given a table with ten blocks (*B* = 10) and 17 tuples per block (*R* = 17) for a total of 170 tuples, compare the cost of reading the whole table sequentially, against the cost of accessing seven random (potentially repeated) tuples in an unknown order. Assume that seek time is 12*ms*, average rotation time is 3*ms* and transferring one block is 0*.*03*ms*. You should consider which is the probability of two tuples accessed consecutively are actually in the same block, but ignore the presence of any cache or buffer pool mechanism.
2. Consider a table stored in HDFS with the following characteristics:
    
    $$
    \begin{align*}
    \text{size}(T) &= 256 \text{ MB} \\
    \text|T| &= 64 \\
    \text{size(row)} &= 4 \text{ MB} \\
    \text{cols}(T) &= 4 \\
    \text{size(cell)} &= 1 \text{ MB} \\
    \text{size(Header)} &= 0 \\
    \text{size(Footer)} &= 0 \\
    \end{align*}
    $$
    
    1. Given the configuration parameters, how much space would you need to store it in a horizontal layout (i.e., Avro)?
        
        $$
        \begin{align*}\text{Size(MetaRow)} &= 0.1\text{MB} \\\text{Size(MetaBody)} &= 0.1\text{MB} \\\end{align*}
        $$
        
    2. Given the configuration parameters, how much space would you need to store it in a hybrid layout (i.e., Parquet)?
        
        $$
        \begin{align*}\text{Size(MetaCol)} &= 0.2\text{MB} \\\text{Size(MetaRowGroup)} &= 0.5\text{MB} \\\text{Size(RowGroup)} &= 8\text{MB} \\\end{align*}
        $$
        
    3. Given the configuration parameters, how much data would you retrieve in a hybrid layout (i.e., Parquet) to project two columns?
        
        $$
        \begin{align*}\text{Size(MetaCol)} &= 0.2\text{MB} \\\text{Size(MetaRowGroup)} &= 0.5\text{MB} \\\text{Size(RowGroup)} &= 8\text{MB} \\\end{align*}
        $$
        
    4. Given the configuration parameters, how much data would you need to retrieve it in a hybrid layout (i.e., Parquet) to select one row (e.g., given its key) if the table is not sorted?
    
    $$
    \begin{align*}\text{Size(MetaCol)} &= 0.2\text{MB} \\\text{Size(MetaRowGroup)} &= 0.5\text{MB} \\\text{Size(RowGroup)} &= 8\text{MB} \\\end{align*}
    $$
    
3. Given a file with *C* chunks and *N* nodes in the cluster, find the probability that HDFS uses all the nodes in case *C* ≥ *N*.
4. Consider an HDFS cluster with 100 data nodes, without replication. If I upload a file with 10 chunks and 10 disk blocks each, answer the following questions and briefly justify your answer:
    1. Which is the maximum number of machines containing data?
    2. Which is the probability of the maximum number of machines actually contain data?
5. Given a file of 3*.*2*GB* stored in an HDFS cluster of 50 machines and containing. $16\space.\space10^5$ key-value pairs in a SequenceFile; estimate the execution time of a Spark job containing a single map transformation and an action storing the results in a file. Explicit any assumption you make and consider also the following parameters:
    - Chunk size: 128*MB* (default)
    - Replication factor: 3 (default)
    - Map function (i.e., the parameter of the transformation) execution time: $10^{-3}$ sec/call (this is **the only cost** you have to consider)
    - Save action execution time: 0sec (do not consider its cost at all)
6. Given a file with 3*.*2*GB* of raw data stored in an HDFS cluster of 50 machines, and containing $16\space.\space10^5$ rows in a **Parquet file**; consider you have a query over an attribute “*A* = constant” and this attribute contains only 100 different and equiprobable values. Assuming any kind of compression has been disabled, explicit any assumption you need to make and give the amount of raw data (i.e., do not count metadata) it would need to fetch from disk.
    - Replication factor: 3 (default)
    - Chunk size: 128*MB* (default)
    - Rowgroup size: 32*MB*
7. Consider a cluster of ten worker machines and a single coordinator, which contains a sequence file of 128MB stored in HDFS with an OS block size of 32KB in all machines and a chunk size of 64MB. On the event of a client retrieving that file, give the **number of control messages** (do not consider data messages) that will travel the network in the case of (a) a client cache miss and in the case of (b) a client cache hit. Briefly justify both numbers.