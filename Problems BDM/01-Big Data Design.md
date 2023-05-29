# 01-Big Data Design

## 1.1 Theoretical questions

1. Briefly explain what “physical independence” is, which is the general position of NOSQL systems on this, and why they take this position.

2. In the framework of the RUM conjecture, name six data structures we saw in the course (not concrete tool implementations) and place them in the corresponding category:
    - Read optimized:
    - Write optimized:
    - Space optimized:
3. Compare a B-tree and an LSM-tree in the context of the **RUM conjecture** (i.e., as an answer to this question, three brief explanations of the form “From the perspective of X, Y-tree is better than Z-tree, because of this and that.” are expected).
    1. a
    2. d
    3. d

## 1.2 Problems

1. Consider:
    1. [’BCN’, POPULATION: {VALUE:’2,000,000’}, REGION: {VALUE:’CAT’}]
    2. [’BCN’, ALL: {VALUE:’2,000,000; CAT’}]
    3. [’BCN’, ALL: {POPULATION:’2,000,000’; REGION:’CAT’}] 
    
    Place {a, b, c} in the table:
    
    |  | Less variable schema | → | More variable schema |
    | --- | --- | --- | --- |
    | Less explicit schema |  |  |  |
    | ⬇️ |  |  |  |
    | More explicit schema |  |  |  |
    
    Name two criteria you would use to choose among them:
    
    1. a
    2. b
2. Consider the following conceptual schema and propose several design alternatives to translate it to a logical representation (e.g., using JSON notation). Then, explain which is the best alternative and why.
 
    
    ![Untitled](01-Big%20Data%20Design%20a287876e227f441b82ebfe4be238dc96/Untitled.png)