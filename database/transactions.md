# Transactions
- A logical unit of processing in a DBMS that entails one or more database access operations
- Purpose: To make sure data in database is consistent
- EX:
  - Alice transfer 100$ to Bob
    - Subtract 100$ from Alice account
    - Add 100$ to Bob account
  - If there is no transaction => action 1 can be successful and action 2 fail => Alice lost 100$

## ACID
- A set of properties ob database transactions intended to guarantee data validity
1. Atomicity (`A`)
    - Each Transaction is treated as a single **unit**. All writes succeed or none of them do
    - Ex: With Above example => if action 2 fail => action 1 fail => Alice not losing her mony
2. Consistency (`C`)
   - Data integrity constraints. Changes made within a transaction are consistent with database constraints.
   - If data get into an illegal state => whole transaction fail
3. Isolation (`I`)
   - Each transaction can pretend that it is the only transaction running on the entire DB => No race conditions, independently of each other
   - ***Race Condition***: 2 or more thread executing concurrently on the same pieces of data
4. Durability (`D`)
    - Committed transactions will remain commited event in the case of system failure 

## 4 Level of Transaction Isolation
1. Read Phenomena
    ![ReadPhenomena](ReadPhenomena.png)
   - Non-Repeatable Read:
     - EX: ![Non-Repeatable Read](read-skew-ex.png)
     - **Solution**: Can use **Right ahead log**
2. Isolation Levels
   ![Transaction Isolation Levels](TransactionIsolationLevels.png)
3. Relationship between isolation levels and read phenomena
   ![Relationship Between isolation levels and read phenomena](RelationshipBetweenIsolationLevelsAndPhenomena.png)