- At least once
- At most once
- Exactly once

# At least once
- No message should be lost
- Acceptable duplicates 
- Moderate Throughput, needs [[ACK]] from [[message broker]]
Kind of like [[TCP]] with it's [[three-way handshake]]

# At most once
- Message could be lost
- No duplicates are acceptable
- High throughput, via [[fire and forget]]
# Exactly once
- Message delivered once
- No message should be lost
- Complex to setup