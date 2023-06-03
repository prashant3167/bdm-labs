# 01-Big Data Design

## 1.1 Theoretical questions

1. Briefly explain what “physical independence” is, which is the general position of NOSQL systems on this, and why they take this position.

- In context of database systems and according to the ANSI-SPARC Architecture, physical independence refers to the term wherein any changes made to the files or data structure shouldn’t affect the way the files are accessed by the system. 
- With the evolution in Big Data demanding high efficiency (i.e., high velocity for increasing volume), architecture of databases tends to move away from traditional RDBMSs as every kind of dataset requires different data models and query languages. Additionally, they are neither declarative nor standardised, hence resulting in loss of physical independence and it is the NOSQL Systems like Key-Value stores, Document Stores, etc. that fall under this category.


2. In the framework of the RUM conjecture, name six data structures we saw in the course (not concrete tool implementations) and place them in the corresponding category:
    - Read optimized: B-tree, Hash, Trie or any other Key-value stores
    - Write optimized: Log-structured merge tree (LSM), PDT
    - Space optimized: Bitmap, Bloom Filter, Sparse Index
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
    | Less explicit schema |  |  | b |
    | ⬇️ |  | c |  |
    | More explicit schema | a |  |  |
    
    Name two criteria you would use to choose among them:
    
    1. How often is the dataset changing (understanding the variability or variety):  If the variability is more, we move towards b else towards a
    2. How much is the space consumed and flexibility to store the data: Space consumed to store b is least while c occupies the most space 
2. Consider the following conceptual schema and propose several design alternatives to translate it to a logical representation (e.g., using JSON notation). Then, explain which is the best alternative and why.
 
    
    ![Untitled](01-Big%20Data%20Design/Untitled.png)

authors: [
{‘a_id’ : ’a1’,     ‘a_name’: ‘a1_name’}, 
{‘a_id’ : ‘a2’,     ‘a_name’: ‘a2_name’}, …] 

books: [
{‘b_id’ : ’b1’,  ‘b_name’: ‘b1_name’}, 
{‘b_id’ : ‘b2’,     ‘b_name’: ‘b2_name’}, …] 

authorBooks : [
        {‘rel_id’: 1,   ‘a_id’ : ‘a1’,  ‘b_id’: ‘b1’}]

Design-1: authors, books, author&Books (all like relational model)
Design-2: authors, books, authorBooks (all books of each author are nested)
Design-3: authors, books, bookAuthors (all authors of each book are nested)
Design-4: books (has nested list of all authors)
Design-5: authors (has nested list of all books) 
Design-6: authors&books (like relational model)
Design-7: authors&books with books having nested list of authors and authors separately

The choice of best alternative depends on the requirements. 
-   Storage space: Design 6 consumes the maximum space as every for m-n relations there are m x n key-value pairs. Design 1 consumes the least space.
-   Types of queries: Design 6 is the best for queries as all the entries are stored separately & we can map it. Design 1 is the worst for queries as we need to traverse through 3 storages and apply joins to check for relations.
