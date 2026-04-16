---
up:
  - "[[003_skills/data-engineering/03 Storage layer|03 Storage layer]]"
down:
prev:
topic: false
question: How replication ensures fault tolerance in HDFS?
---
# How replication ensures fault tolerance in HDFS?

> [!Summary] Summary
> 
> - HDFS creates a replication of 3 by default i. e each block is stored in 3 nodes
> - Hadoop's rack aware policy replicates 1st copy at the local, 2nd copy in a different rack and 3rd copy is stored in remote rack
> - Data node send heartbeat every few seconds to the name node. If any name node fails to send then the name node marks the node dead and starts replicating. Data node also send list of all the blocks they are currently storing.
> - When a file is created HDFs creates checksum for every block
> - When a client reads the data, it recalculates the checksum. if checksum does not match the client informs name node about corrupt block. The name node deletes the corrupt block and replace with fresh healthy copy

---
- Replication is the backbone of HDFS's reliability. 
- Since Hadoop is designed to run on "commodity hardware" (standard, affordable servers that are prone to failure), it assumes that hardware **will** fail and plans for it automatically.

---

## 1. The Power of Three: Default Replication

By default, HDFS uses a **Replication Factor of 3**. This means every single block of data is stored in three different physical locations.

### Rack Awareness
HDFS doesn't just throw these copies at random servers; it uses a **Rack Aware** policy to ensure maximum safety:
1. **First Copy:** Placed on the local node where the client is performing the write.
2. **Second Copy:** Placed on a different node within a **different rack**.
3. **Third Copy:** Placed on a different node within that **different remote rack**.

> **Why this matters:** If an entire rack loses power or its network switch fails, you still have a copy sitting safely in a completely different rack.

## 2. Heartbeats and Block Reports
The **NameNode** (the master) constantly monitors the health of the **DataNodes** (the workers) through two mechanisms:
- **Heartbeats:** Every few seconds, DataNodes send a "heartbeat" signal. If the NameNode stops receiving these from a specific node, it marks that node as "dead."
- **Block Reports:** Periodically, DataNodes send a list of all the blocks they are currently storing.

## 3. The Self-Healing Process
When a DataNode fails, HDFS triggers an automatic **Self-Healing** routine to restore fault tolerance without any human intervention:
1. **Detection:** The NameNode realizes a DataNode is down because the heartbeats stopped.
2. **Identification:** The NameNode looks at its metadata to see which blocks were stored on that failed node.
3. **Under-Replication Check:** The NameNode realizes those specific blocks now only have 2 copies instead of the required 3.
4. **Redistribution:** The NameNode instructs other healthy DataNodes that have copies of those blocks to replicate them to new, functional nodes.
5. **Restoration:** Once the new copies are created, the replication factor returns to 3, and the system is fully fault-tolerant again.

## 4. Integrity Protection (Checksums)
Fault tolerance isn't just about server crashes; it’s also about **bit rot** (data corruption over time).
- When a file is created, HDFS calculates a **checksum** for every block.
- When a client reads the data, it recalculates the checksum.
- If the checksums don't match, the client informs the NameNode that the block is corrupt.
- The NameNode then deletes the corrupt copy and replaces it with a fresh, healthy copy from one of the other two replicas.


