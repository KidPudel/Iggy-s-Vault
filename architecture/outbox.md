Pattern used in distributed systems to ensure reliable message delivery and data consistency between db and [[message broker]]
It works by storing events in `outbox` table in db as part of the same transaction that is happening in system's change
It then forwarding messages from this table to the [[message broker]] to ensure that no message will be lost
