Isolation is for managing level of access to the changes that are being made to the same thing during the process of different writes to avoid conflicts 

# Serializable
The **gold standard** is "serializable" (**not** [[serialization]] in terms of converting).
When you have multiple transaction happening concurrently/at the same time, **the resulting state of DB** is set to the result of whatever state could be achieved by running those [[transactions]] **sequentially**, and it doesn't matter in what order

> After the gold standard is set, consensus suggested new isolations, because of the performance.

They released a list of "[[anomalies]]", occurred without Serializable

| Isolation Level  | Dirty Reads  | Non-repeatable Reads | Phantom Reads |
| ---------------- | ------------ | -------------------- | ------------- |
| Read Uncommitted | Possible     | Possible             | Possible      |
| Read Committed   | Not Possible | Possible             | Possible      |
| Repeatable Reads | Not Possible | Not Possible         | Possible      |
| Serializable     | Not Possible | Not Possible         | Not Possible  |
