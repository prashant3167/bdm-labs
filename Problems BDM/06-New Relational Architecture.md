# 06-New Relational Architecture

## 6.1 Theoretical questions

1. Briefly explain the concept of associativity in the context of memory usage. Identify the benefit of high (respectively low) associativity.

## 6.2 Problems

1. Assume you have a SanssouciuDB table T stored **row-wise**, which occupies 300 blocks and contains tuples with three variable length attributes [A, B, C] (underlined attribute is declared to be the primary key of the table). Assume there is **no index** and give the **average cost** of each query assuming accessing one block is one second.
    - SELECT A,B,C FROM T WHERE A=x; (being x a constant)
    - SELECT SUM(B) FROM T;
2. Assume you have a SanssouciuDB table T stored **column-wise**, which occupies 300 blocks and contains tuples with three attributes [A, B, C] (underlined attribute is declared to be the primary key of the table). Assume there is **no index**, storage of each attribute uses exactly the same space after compression (i.e., 100 blocks), and run length encoding has been applied for non-key attributes storing ending row position per run. Give the **average cost** of each query assuming accessing one block is one second and explicit any other assumption you make.
    - SELECT A,B,C FROM T WHERE A=x; (being x a constant)
    - SELECT SUM(B) FROM T;
3. Assume that there is a table T with attributes [A,B,C] (the underlined attribute is the primary key of the table), which occupies 300 disk blocks of 8KB each, with 100bytes per row. All three attributes require the same space (even in case of compression). Supposing that there is not any index, which would be the **minimum amount of memory required to process** (do not consider the memory required to store the final output of the query) the following query in either row storage or column storage (briefly **justify** your answer and explicit any assumption you make).
    
    SELECT A FROM T WHERE B=’x’ AND C=’y’;
    
    - Row storage
    - Column storage
4. Without considering compression and assuming a disk block size of 8KB, is there any case where a query retrieving all tuples, but only half of the equally-sized attributes of a relational table performs better in **row storage** without any kind of vertical partitioning than in columnar storage? Numerically justify your answer.
5. Represent the given column with dictionary and run-length encoding storing end row position.
    
    ![Untitled](06-New%20Relational%20Architecture/Untitled.png)
    
6. Given the two columns of a table represented with run-length encoding using dictionary, give the position of the row(s) that fulfill the predicate ”Charlie AND Beta”. Briefly explain how a column store would obtain them.
    
    ![Untitled](06-New%20Relational%20Architecture/Untitled%201.png)