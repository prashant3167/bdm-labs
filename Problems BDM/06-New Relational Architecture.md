# 06-New Relational Architecture

## 6.1 Theoretical questions

1. Briefly explain the concept of associativity in the context of memory usage. Identify the benefit of high (respectively low) associativity.

## 6.2 Problems

1. Assume you have a SanssouciuDB table T stored **row-wise**, which occupies 300 blocks and contains tuples with three variable length attributes [<u>A</u>, B, C] (underlined attribute is declared to be the primary key of the table). Assume there is **no index** and give the **average cost** of each query assuming accessing one block is one second.
    - `SELECT A,B,C FROM T WHERE A=x; # (being x a constant)`

    300s would be the cost for full sequencial cost for worst case if we've to read all. But it is possible that we can find the solution anywhere in between therefore the average cost is average between best and worst case.
    $$
    \frac{{1 + 300}}{2} = 150.5s
    $$


    - `SELECT SUM(B) FROM T;`

    Since we are aggregating all records therefore the answer is 300s.

2. Assume you have a SanssouciuDB table T stored **column-wise**, which occupies 300 blocks and contains tuples with three attributes [<u>A</u>, B, C] (underlined attribute is declared to be the primary key of the table). Assume there is **no index**, storage of each attribute uses exactly the same space after compression (i.e., 100 blocks), and run length encoding has been applied for non-key attributes storing ending row position per run. Give the **average cost** of each query assuming accessing one block is one second and explicit any other assumption you make.
    - `SELECT A,B,C FROM T WHERE A=x; # (being x a constant)`

        Same as before the average of best(1s) and worst case(100s):

        $$
        \frac{{1 + 100}}{2} = 50.5s
        $$

        But there could be a possibility while mapping the column B and C to A after filtering hence we can add a safe margin of 1 more in which case the mapping would be either at upper or down position by at most 1 position and the revised equation would look like:

        $$
        \frac{{1 + 100}}{2} + 1(\text{for Col }B) + 1(\text{for Col }C) = 52.5s
        $$

    - `SELECT SUM(B) FROM T;`

        $$
        100s
        $$



3. Assume that there is a table T with attributes [<u>A</u>,B,C] (the underlined attribute is the primary key of the table), which occupies 300 disk blocks of 8KB each, with 100bytes per row. All three attributes require the same space (even in case of compression). Supposing that there is not any index, which would be the **minimum amount of memory required to process** (do not consider the memory required to store the final output of the query) the following query in either row storage or column storage (briefly **justify** your answer and explicit any assumption you make).


    
    `SELECT A FROM T WHERE B=’x’ AND C=’y’;`
    
    - Row storage

        8kb is enough, because we process one block at a time in memmory which is 8kb. 

    - Column storage

        We need three blocks (24kb) since we need to filter B and C. The reason we need the third block is we've to retrieve the A as part of selection for which we need memory.

4. Without considering compression and assuming a disk block size of 8KB, is there any case where a query retrieving all tuples, but only half of the equally-sized attributes of a relational table performs better in **row storage** without any kind of vertical partitioning than in columnar storage? Numerically justify your answer.



5. Represent the given column with dictionary and run-length encoding storing end row position.
    
    ![Untitled](06-New%20Relational%20Architecture/Untitled.png)

    Dictionary: A, B, C
    
    End Row: 1, 4, 5, 7 (Assuming that the rows are stored in the form of 0-index)
    
    Code: 1, 2, 3, 1

    
6. Given the two columns of a table represented with run-length encoding using dictionary, give the position of the row(s) that fulfill the predicate ”Charlie AND Beta”. Briefly explain how a column store would obtain them.
    
    ![Untitled](06-New%20Relational%20Architecture/Untitled%201.png)

    Column 1 – Bravo, Bravo, NULL, NULL, NULL, Charlie, Charlie, Charlie
    Column 2 – Alpha, NULL, NULL, NULL, NULL, NULL, Beta, Beta
    
    Row Positions obtained in query predicate: 6 and 7.
    
    Explanation – A columnar store database will first convert the number of values stored to rows ending positions, thus we will get [1, 4, 7] and [0, 5, 7] for columns 1 and 2 respectively. Then, checking for predicate Charlie (in the range of 5-7) and Beta (in the range of (6-7) => ANDing gets us rows 6-7, which is our required answer.

