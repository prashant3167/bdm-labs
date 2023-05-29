# 07-Distributed Processing Frameworks

## 7.1 **MapReduce in use**

### 7.1.1 Theoretical questions

1. Classify a MapReduce system as either query-shipping (i.e., the evaluation of the query is delegated to the site where the data are stored), data-shipping (i.e., the data are moved from the stored site to the site executing the query) or hybrid (i.e., both query- and data-shipping). Clearly justify your answer.

### 7.1.2 Problems

1. Consider the following implementation of the Cartesian Product with MapReduce ($\oplus$ stands for concatenation) and answer the questions accordingly.
    
    ![Untitled](07-Distributed%20Processing%20Frameworks/Untitled.png)
    
    1. Which is the relationship of *D* with parallelism and scalability, if any?
    2. Which is the optimal value for *D*?
2. Provide the MapReduce pseudo-code implementation of the following Relational operators. You can use “$\oplus$” symbol for concatenation, $"pr\space j_{a_{i_1} ... a_{i_n}}(t)"$ to get attributes $"a_{i_1}...a_{i_n}"$ in *t*, $crossproduct(S_1, S_2)$ to perform the cross product of two sets of rows, and assume the “key” parameter contains the PK of the table and the “value” one all the others.
    - Aggregation (*γA,f*(*B*)(*T* ))
    - Selection (*σP* (*T* ))
    - Join (*T ⨝ $_{T.A=S.B}S$*)
    - Union (T ∪ S)
    - Difference (T \ S)
    - Intersection (T ∩ S)
3. In relational algebra, the antijoin operator (*▷*) is defined as the complement of the semi-join on the primary keys (PKs). Formally, assuming *A* and *B* are the PKs of *R* and *S*, respectively, then *R▷S* = *R \ R* ⋉*A*=*B* *S*. Provide the MapReduce pseudo-code implementation of the antijoin operator. Assume the existence of the operator $\oplus$ to concatenate strings, $prj_{att}(s)$ to project attribute *att* from the tuple *s*, and *input*(*s*) to decide the origin (i.e., *R* or *S*) from a tuple *s*.

## 7.2 **MapReduce internals**

### 7.2.1 Problems

1. Assume the following MapReduce program:
    
    ```java
    public void map(LongWritable key, Text value) {
    		String line = value.toString();
    		StringTokenizer tokenizer = new StringTokenizer(line);
    		while (tokenizer.hasMoreTokens()) {
    		write(new Text(tokenizer.nextToken()), new IntWritable(1));
    		}
    }
    public void reduce(Text key, Iterable<IntWritable> values) {
    		int sum = 0;
    		for (IntWritable val : values) {
    		sum += val.get();
    		}
    		write(key, new IntWritable(sum));
    }
    ```
    
    Consider the following data set:
    
    - Block0: ”a b b a c | c d c e a”
    - Block1: ”a b d d a | b b c c f”
    
    Simulate the execution of the MapReduce code given the following configuration:
    
    - The **map** and **reduce functions** are those of the wordcount
        - The **combine** function shares the implementation of the reduce
    - One **Split** is one block
    - The ”|” divides the **records** inside each block
        - We have two records per block
    - Hadoop is configured with the parameter *dfs.replication=1*
    - We can keep four pairs [key,value] per **spill**
    - We have two **mappers** and two **reducers**
        - *Machine*0, contains block0, runs mapper0 and reducer0
        - *Machine*1, contains block1, runs mapper1 and reducer1
    - The hash function used to shuffle data to the reducers uses the correspondence:
        - {b,d,f} → 0
        - {a,c,e} → 1
    
    Fill the gaps in each step (numbers correspond to the phase in the MapReduce algorithm):
    
    1. *Machine*0 contains · · · block(s).
        
        *Machine*1 contains · · · block(s).
        
    2. We keep · · · replica(s) (including the master copy) per block.
    3. We have · · · split(s) per machine.
    4. Mapper0 reads · · · records. 
    Mapper1 reads · · · records.
    5. Memory content during each spill in *Machine*0:
        
        
        | Spill1 | Spill2 | Spill3 | Spill4 |
        | --- | --- | --- | --- |
        | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] |
        
        Spills in Machine1:
        
        | Spill1 | Spill2 | Spill3 | Spill4 |
        | --- | --- | --- | --- |
        | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] | [ , ] [ , ]
        [ , ] [ , ] |
    6. 
        1. Partitions in *Machine*0:
            
            
            | Spill1 | Spill2 | Spill3 | Spill4 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | Partition0 | Partition1 | Partition0 | Partition1 | Partition0 | Partition1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ , ] [ , ]
            [ , ] [ , ] | [ , ] [ , ]
            [ , ] [ , ] | [ , ] [ , ]
            [ , ] [ , ] | [ , ] [ , ]
            [ , ] [ , ] | [ , ] [ , ]
            [ , ] [ , ] | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
            
            Partitions in Machine1:
            
            | Spill1 | Spill2 | Spill3 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | Partition0 | Partition1 | Partition0 | Partition1 | Partition0 | Partition1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
        2. Partitions in *Machine*0:
            
            
            | Spill1 | Spill2 | Spill3 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | Partition0 | Partition1 | Partition0 | Partition1 | Partition0 | Partition1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
            
            Partitions in Machine1:
            
            | Spill1 | Spill2 | Spill3 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | Partition0 | Partition1 | Partition0 | Partition1 | Partition0 | Partition1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
            | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] | [ | , ] [ | , ] | [ | , | ] [ | , | ] |
        3. Files in *Machine*0:
            
            
            | File0.0 | File0.1 | File0.2 |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] |
            | File0.3 | File0.4 | File0.5 |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] |
            
            Files in Machine1:
            
            | File1.0 | File1.1 | File1.2 |  |  |  |  |  |  |  |  |  |  |  |  |
            | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
            | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] |
            | File1.3 | File1.4 | File1.5 |  |  |  |  |  |  |  |  |  |  |  |  |
            | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] |
    7. Merges in *Machine*0:
        
        
        | Merge0 | Merge1 |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] |
        
        Merges in Machine1:
        
        | Merge0 | Merge1 |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] |
    8. Files in *Machine*0:
        
        
        | File0.0 | File0.1 |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] |
        
        Files in Machine1:
        
        | File1.0 | File1.1 |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] | [ | , ] [ | , ] [ | , ] [ | , ] [ | , ] |
    9. Reducer0 reads · · · from *Machine*0 and · · · from *Machine*1. 
    Reducer1 reads · · · from *Machine*0 and · · · from *Machine*1.
    10. Merges in *Machine*0:
        
        
        | Merge0 | Merge1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] | [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] |
        
        Merges in Machine1:
        
        | Merge0 | Merge1 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] | [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] [ | ,{ | }] |
    11. Reduce function is executed · · · times in *Machine*0. 
    Reduce function is executed · · · times in *Machine*1.
    12. Files in *Machine*0:
        
        
        | Output0 |  |  |  |  |
        | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] |
        
        Files in Machine1:
        
        | Output1 |  |  |  |  |
        | --- | --- | --- | --- | --- |
        | [ | , ] [ | , ] [ | , ] [ | , ] |
2. Let’s suppose that we have a cluster of 100 machines and a MapReduce job with 1.000.000 key-value pairs in the input that generate 100.000 pairs in the output. Assume that both map and reduce functions generate one pair in the output per call. Assuming the reduce function is commutative and associative, is it worth to use the combine function? Briefly justify your answer.

## 7.3 **Spark in use**

### 7.3.1 Problems

1. Consider a file (*wines.txt*) containing the following data:
    
    ```
    wines.txt
    type 1,2.064254
    type 3,2.925376
    type 2,2.683955
    type 1,2.991452
    type 2,2.861996
    type 1,2.727688
    ```
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, just the kind of operation and main parameters) you’d need to retrieve the minimum value per type. Do not use SQL and minimize the use of other Python libraries or code. Save the results in *output.txt*.
    
2. Consider two files containing the following data:
    
    ```xml
    Employees.txt
    EMP1;CARME;400000;MATARO;DPT1
    EMP2;EUGENIA;350000;TOLEDO;DPT2
    EMP3;JOSEP;250000;SITGES;DPT3
    EMP4;RICARDO;250000;MADRID;DPT4
    EMP5;EULALIA;150000;BARCELONA;DPT5
    EMP6;MIQUEL;125000;BADALONA;DPT5
    EMP7;MARIA;175000;MADRID;DPT6
    EMP8;ESTEBAN;150000;MADRID;DPT6
    
    Departments.txt
    DPT1;DIRECCIO;10;PAU CLARIS;BARCELONA
    DPT2;DIRECCIO;8;RIOS ROSAS;MADRID
    DPT3;MARKETING;1;PAU CLARIS;BARCELONA
    DPT4;MARKETING;3;RIOS ROSAS;MADRID
    DPT5;VENDES;1;MUNTANER;BARCELONA
    DPT6;VENDES;1;CASTELLANA;MADRID
    ```
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, just the kind of operation and main parameters) you’d need to retrieve for each employee his/her department information. Do not use SQL and minimize the use of other Python libraries or code. Save the results in *output.txt*.
    
3. Consider an error log file (*log.txt*) like the one bellow:
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, just the kind of operation and main parameters) you’d need to retrieve the lines corresponding to both *errors* and *warnings*, but adding *Important:* at the beginning of those of errors (i.e., only *errors*). Do not use SQL and minimize the use of other Python libraries or code. Save the results in *output.txt*.
    
    ```java
    log.txt 
    20150323;0833;ERROR;Oracle
    20150323;0835;WARNING;MySQL
    20150323;0839;WARNING;MySQL
    20150323;0900;WARNING;Oracle
    20150323;0905;ERROR;MySQL
    20150323;1013;OK;Oracle
    20150323;1014;OK;MySQL
    20150323;1055;ERROR;Oracle
    ```
    
4. Given two files containing the following kinds of data:
    
    ```java
    Employees.txt with fields: EmployeeID; EmployeeName; YearlySalary; CityOfResidence; SiteOfWork
    		EMP4;RICARDO;250000;MADRID;DPT4
    		EMP5;EULALIA;150000;BARCELONA;DPT5
    		EMP6;MIQUEL;125000;BADALONA;DPT5
    		EMP7;MARIA;175000;MADRID;DPT6
    		EMP8;ESTEBAN;150000;MADRID;DPT6
    		...
    
    Departments.txt with fields: SiteID; DepartmentName; StreetNumber; StreetName; City 
    		DPT1;DIRECCIO;10;PAU CLARIS;BARCELONA
    		DPT2;DIRECCIO;8;RIOS ROSAS;MADRID
    		DPT3;MARKETING;1;PAU CLARIS;BARCELONA
    		DPT4;MARKETING;3;RIOS ROSAS;MADRID
    		...
    ```
    
    Consider the following **PySpark code** and answer the questions bellow.
    
    ```java
    source1 = spark.read.format(”csv”).load(”employees.txt”, header=’false’, inferSchema=’true’, sep=”;”)
    source2 = spark.read.format(”csv”).load(”departments.txt”, header=’false’, inferSchema=’true’, sep=”;”)
    A = source1.toDF(”eID”,”eName”,”eSalary”,”eCity”,”eDpt”,”eProj”)
    B = source2.toDF(”dID”,”dArea”,”dNumber”,”dStreet”,”dCity”)
    C = A.select(A.eCity.alias(”city”))
    D = B.select(”dArea”)
    E = D.crossJoin(C)
    F = B.select(”dArea”,B.dCity.alias(”city”))
    G = E.subtract(F)
    H = G.select(”dArea”) 
    result = D.subtract(H)
    ```
    
    1. State in natural language the corresponding query it would answer?
    2. Clearly indicate any mistake or improvement you can fix/make in the code? For each of them give (1) the line number, (2) pseudo-code to implement the fix, and (3) brief rationale.
5. Given two files containing the following kinds of data:
    
    ```java
    Employees.txt with fields: EmployeeID; EmployeeName; YearlySalary; CityOfResidence; SiteOfWork
    		EMP1;RICARDO;250000;MADRID;DPT1
    		EMP2;EULALIA;150000;BARCELONA;DPT2
    		EMP3;MIQUEL;125000;BADALONA;DPT3
    		EMP4;MARIA;175000;MADRID;DPT4
    		EMP5;ESTEBAN;150000;MADRID;DPT3
    		...
    
    Departments.txt with fields: SiteID; DepartmentName; StreetNumber; StreetName; City
    		DPT1;DIRECCIO;10;PAU CLARIS;BARCELONA
    		DPT2;DIRECCIO;8;RIOS ROSAS;MADRID
    		DPT3;MARKETING;1;PAU CLARIS;BARCELONA
    		DPT4;MARKETING;3;RIOS ROSAS;MADRID
    		...
    ```
    
    Give a sequence of Spark operations in pseudo-code (resembling PySpark) to obtain for each city where employees that work in a site of a department in Barcelona live, the sum of the salaries of those employees. The result for the exemplary data would be:
    
    ```java
    MADRID;400000
    BADALONA;125000
    ...
    ```
    
6. Consider three files containing the following kinds of data:
    
    ```java
    Employees.txt
    EMP1,CARME,400000,MATARO,DEPT1,PROJ1
    EMP2,EULALIA,150000,BARCELONA,DEPT2,PROJ1
    EMP3,MIQUEL,125000,BADALONA,DEPT1,PROJ3
    
    Projects.txt
    PROJ1,IBDTEL,TV,1000000
    PROJ2,IBDVID,VIDEO,500000
    PROJ3,IBDTEF,TELEPHONE,200000
    PROJ4,IBDCOM,COMMUNICATIONS,2000000
    
    Departments.txt
    DEPT1,MANAGEMENT,10,PAU CLARIS,BARCELONA
    DEPT2,MANAGEMENT,8,RIOS ROSAS,MADRID
    DEPT4,MARKETING,3,RIOS ROSAS,MADRID
    ```
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, but just the kind of operation and main parameters) you would need to obtain the departments with all employees assigned to the same project. The result must include department number. Save the results in output.txt. In the previous example, the result should be *DEPT2* and *DEPT4*.
    
7. Consider two files containing the following kinds of data:
    
    ```java
    Employees.txt
    EMP4;RICARDO;250000;MADRID;DPT4
    EMP5;EULALIA;150000;BARCELONA;DPT5
    EMP6;MIQUEL;125000;BADALONA;DPT5
    EMP7;MARIA;175000;MADRID;DPT6
    EMP8;ESTEBAN;150000;MADRID;DPT6
    
    Departments.txt
    DPT1;DIRECCIO;10;PAU CLARIS;BARCELONA
    DPT2;DIRECCIO;8;RIOS ROSAS;MADRID
    DPT3;MARKETING;1;PAU CLARIS;BARCELONA
    DPT4;MARKETING;3;RIOS ROSAS;MADRID
    ```
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, but just the kind of operation and main parameters) you’d need to retrieve the list of department IDs for those departments with workers from all cities where there are employees. Save the results in output.txt.
    
8. Consider two files containing the following kinds of data:
    
    ```java
    Employees.txt
    EMP1;RICARDO;250000 C;MADRID;SITE2
    EMP2;EULALIA;150000 C;BARCELONA;SITE1
    EMP3;MIQUEL;125000 C;BADALONA;SITE3
    EMP4;MARIA;175000 C;MADRID;SITE2
    EMP5;ESTEBAN;150000 C;MADRID;SITE4
    
    Departments.txt
    SITE1;DPT.MANAGEMENT;FLOOR10;ST.PAU CLARIS;BARCELONA
    SITE2;DPT.MANAGEMENT;FLOOR8;ST.RIOS ROSAS;MADRID
    SITE3;DPT.MARKETING;FLOOR1;ST.PAU CLARIS;BARCELONA
    SITE4;DPT.MARKETING;FLOOR1;ST.RIOS ROSAS;MADRID
    SITE5;DPT.MARKETING;FLOOR5;ST.MARTI PUJOL;BADALONA
    ```
    
    Provide the ordered list of Spark operations (no need to follow the exact syntax, but just the kind of operation and main parameters) you’d need to retrieve the list of department IDs for those departments with sites in all cities where employees live (these employees can be even from other departments). Save the results in output.txt. In the previous example, the result should be *DPT.MANAGEMENT*, because it has sites in all the three cities where there are employees (i.e., MADRID, BARCELONA and BADALONA). However, *DPT.MANAGEMENT* should not be in the result, because it does not have any site in BADALONA, where EMP3 lives.
    
9. Consider three files relating to a bibliographic database: author.csv relates authors with papers (you may assume that author names are unique, that authors have one or more papers, and that papers have one or more authors); title.csv gives the title of a paper (you may assume a paper has one title, but one title may be shared by many papers); and citation.csv indicates which papers cite which other papers (you may assume that each paper cites at least one other paper, that a paper may be cited zero or more times, and that a paper cannot cite itself).
    
    ![Untitled](07-Distributed%20Processing%20Frameworks/Untitled%201.png)
    
    The headers are shown for illustration here. They do not need to be considered.
    
    The count of self-citations for an author *A*, denoted self(*A*), is defined as the number of citation pairs (*P*1*, P*2) where *A* is an author of both. The count of citations given by an author *A*, denoted give(*A*), is the count of citation pairs (*P*1*, P*2) such that *A* is an author of *P*1. The count of citations received by *A*, denoted receive(*A*), is the count of citation pairs (*P*1*, P*2) where *A* is an author of *P*2. The ratio of self-citations to all citations given and received are then defined, respectively, as $\frac{self(A)}{give(A)}$ and $\frac{self(A)}{receive(A)}$ . In case that receive(A) = 0, you should omit the author A from the results (note that give(A) cannot be 0 as an author must have at least one paper and a paper must have at least one citation). We provide an example output for the input data:
    
    | AUTHOR | SELFGIVERATIO | SELFRECEIVERATIO |
    | --- | --- | --- |
    | C. Gutierrez | 1.000 | 1.000 |
    | J. Perez | 0.333 | 1.000 |
    | R. Angles | 0.000 | 0.000 |
    | … | … |  |
    
    Headers are only shown for illustration. We will use Apache Spark to perform the analysis and compute the output. You should not assume any ordering of the input files. You do not need to order the output file in any particular way.
    
    Given this input and desired output, design a Spark process to complete the required processing. In particular, you should draw the high-level DAG of operations that the Spark process will perform, detailing the sequence of transformations and actions. You should briefly describe what each step does, clearly indicating which steps are transformations and which are actions. You should also indicate which RDDs are virtual and which will be materialized. You should use caching if appropriate. You should provide details on any functions passed as arguments to the transformations/actions you use.
    

## 7.4 **Spark internals**

### 7.4.1 Theoretical questions

1. What indicates to Spark query optimizer the end of a stage and the beginning of the next one?

### 7.4.2 Problems

1. Considering the file and result example, briefly indicate the problems you find in the Spark code below (if any), and **modify the code** to fix them (if needed).
    
    ```java
    Employees.txt
    EMP1;CARME;40000;MATARO;DPT1;PROJ1
    EMP2;EUGENIA;35000;TOLEDO;DPT2;PROJ1
    EMP3;JOSEP;25000;SITGES;DPT3;PROJ2
    EMP4;RICARDO;25000;MADRID;DPT4;PROJ2
    EMP5;EULALIA;15000;BARCELONA;DPT5;PROJ2
    EMP6;MIQUEL;12500;BADALONA;DPT5;PROJ3
    EMP7;MARIA;17500;MADRID;DPT6;PROJ3
    EMP8;ESTEBAN;15000;MADRID;DPT6;PROJ3
    
    Expected result
    [ PROJ1,[ EUGENIA, 35000, 37500.0 ] ]
    [ PROJ2,[ EULALIA, 15000, 21666.6 ] ]
    [ PROJ3,[ MIQUEL, 12500, 15000.0 ] ]
    
    rawEmps = sc.textFile(r"Employees.txt")
    emps = rawEmps.map(**lambda** l: tuple(l.split(";"))).cache()
    RDD1 = emps.map(**lambda** t: (t[5], (int(t[2]), 1)))
    RDD2 = RDD1.reduceByKey(**lambda** t1, t2: (t1[0] + t2[0])/(t1[1] + t2[1]))
    RDD3 = emps.map(**lambda** t: (t[5], t))
    RDD4 = RDD3.join(RDD2)
    RDD5 = RDD4.filter(**lambda** t: int(t[1][0][2])<t[1][1])
    RDD6 = RDD5.map(**lambda** t: (t[0],(t[1][0][1],t[1][0][2],t[1][1])))
    Result = RDD6.sortByKey()
    ```
    
2. Consider the following pipeline. This pipeline runs in a 4-machine cluster with HDFS to store the files and Spark to execute it. File1 and File2 are distributed in the cluster in 6 chunks each.
    
    ```java
    RddF1 = sc.textFile("...file1.txt")
    RddF2 = sc.textFile("...file2.txt")
    Rdd2 = RddF1.mapToPair(lambda s: (s.split(";")[0], s.split(";")[1-2]))
    Rdd3 = Rdd2.groupByKey()
    Rdd4 = Rdd3.mapValues(f1)
    Rdd5 = RddF2.mapToPair(lambda s: (s.split(";")[0], s.split(";")[1-2]))
    Rdd6 = Rdd5.groupByKey()
    Rdd7 = Rdd6.mapValues(f2)
    Rdd8 = Rdd4.join(Rdd7)
    Rdd8.save("...file3.txt")
    ```
    
    1. How many stages will the scheduler generate for this pipeline? (Justify your answer)
    2. How many tasks will be generated within each stage? (Justify your answer)
3. Consider the legacy code written in MapReduce that specifies a map(), combine() and reduce() operations. This job reads from a text file $f_1$. The combine and reduce operations coincide and you can assume all functions are correct. Write a Spark pipeline **equivalent to the MapReduce** job. Use fmap and freduce to refer to the code executed inside the map and combine / reduce operations. You can parametrize the fmap and freduce functions but resulting in minimal code adaptation.
4. Assume we have a **MongoDB** collection in a distributed cluster, which contains prices of apartments without any secondary index. Such collection is big enough not to completely fit in memory. We want to **use Spark** to compute the standard deviation per neighborhood. Clearly identify the most efficient option and briefly justify the choice (it is **not necessary to provide the Spark code**).
    1. Use Spark only to push the query to MongoDB aggregation framework and simply get the result.
    2. Push only some of the operations to MongoDB aggregation framework and the run the rest in Spark.
    3. Load the whole collection to an RDD and perform all computations in Spark.