
```
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```


Money Transfer -> what if debit succeeds but credit fails -> it should not happen

---------

# Atomicity

Atomicity : All or nothing : 
A transaction either completes completely, or none of its changes are applied.

SUCCESS → COMMIT
FAILURE → ROLLBACK

```
BEGIN;

UPDATE account
SET balance = balance - 10000
WHERE account_id = 'A';

UPDATE account
SET balance = balance + 10000
WHERE account_id = 'B';

COMMIT;
```


# Consistency
"Valid state → Valid state"

All transaction should move the state from one valid state to another.

To be consistent, the transactions must preserve constraints such as:

- primary keys
- foreign keys
- unique constraints
- check constraints
- business rules implemented by the transaction/application

Example : 
If overdraft is not allowed, then no transaction should leave the negative account balance
You should not be able to open a account without customer.


# Isolation 
"Concurrent transactions shouldn't interfere incorrectly"



Two concurrent transaction both trying to debit 7K from a account with balance of 10K.

Isolation controls how concurrent transactions interact.

Some isolation levels are 
```
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

read uncommitted -> dirty reads
read committed -> 
repeatable read -> 
serializable -> provides strong correctness but can reduce concurrency

# Durability

committed means it survives

The database typically achieves this through mechanisms such as:

- transaction logs
- write-ahead logging (WAL)
- redo logs
- durable storage
- replication
- checkpoints

The exact mechanism depends on the database.






