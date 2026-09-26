
https://doc.akka.io/libraries/akka-core/current/split-brain-resolver.html#docs


## Akka Cluster — Split-Brain Resolver (SBR)

Akka Cluster uses the **Split-Brain Resolver (SBR)** to automatically decide which side of a network partition should remain active when the cluster splits.

### Configuration

```
akka.cluster.downing-provider-class =
  "akka.cluster.sbr.SplitBrainResolverProvider"
```

The active strategy is configured using:

```
akka.cluster.split-brain-resolver.active-strategy
```

---

## SBR Strategies

### 1. `keep-majority` — Default

Keeps the side containing the **majority of the cluster members** and downs the minority side.

**Best for:** Dynamic cluster sizes where maintaining the largest healthy partition is desirable.

**Example:**

```
5-node cluster

Partition:
  Side A → 3 nodes → KEEP
  Side B → 2 nodes → DOWN
```

---

### 2. `static-quorum`

Keeps a partition only if it contains at least the configured **quorum size**. Partitions below the quorum are downed.

**Best for:** Fixed-size clusters where a minimum number of nodes is required for safe operation.

**Example:**

```
5-node cluster
quorum-size = 3

Side A → 3 nodes → KEEP
Side B → 2 nodes → DOWN
```

---

### 3. `keep-oldest`

Keeps the partition containing the **oldest cluster member** and downs the other side.

This is particularly useful when a **Cluster Singleton** is running on the oldest node and you want to ensure that only one partition continues operating the singleton.

**Best for:** Systems where the oldest member has special responsibilities.

---

### 4. `down-all`

Downs **all nodes** when a split-brain situation is detected.

**Best for:** Highly unstable network environments where choosing a surviving partition could result in unsafe or inconsistent behavior.

The system effectively chooses:

```
Safety > Availability
```

---

### 5. `lease-majority`

Uses an external **distributed lease/lock** to determine which partition is allowed to remain active.

For example, the lease may be backed by an external system such as Kubernetes.

Only one side can successfully acquire the lease:

```
Partition

Side A ──→ acquire lease ✓ → KEEP

Side B ──→ acquire lease ✗ → DOWN
```

**Best for:** Environments where an external coordination mechanism can provide a stronger guarantee than membership information alone.

---

# Key Tuning Settings

### `stable-after`

Defines how long the cluster membership/reachability information must remain stable before SBR makes a decision.

The value should generally increase with cluster size because larger clusters can take longer to converge.

**Example guidance:**

```
5 nodes     → ~7 seconds
1000 nodes  → ~30 seconds
```

The exact value should be tuned based on the cluster's failure-detection and network characteristics.

**Purpose:**

> Avoid making a split-brain decision while the cluster is still experiencing transient network instability.

---

### `down-all-when-unstable`

Provides a safety mechanism for prolonged instability.

If SBR cannot reach a stable decision within the configured time window, it downs all nodes.

Conceptually:

```
Network instability
        ↓
 Wait for stable-after
        ↓
Still unstable?
        ↓
Wait additional timeout
        ↓
No safe decision
        ↓
DOWN ALL
```

This prevents the cluster from remaining indefinitely in an uncertain state.

---

### `down-all-when-indirectly-connected`

Controls behavior when nodes are **indirectly connected** rather than being cleanly separated into two independent partitions.

If too many nodes would need to be downed because of indirect/unreliable connectivity, SBR can choose to **down all nodes** rather than risk an unsafe decision.

This is another **safety-over-availability** mechanism.

---

# Strategy Comparison

|Strategy|Decision based on|Typical use|
|---|---|---|
|`keep-majority`|Majority of members|Dynamic clusters|
|`static-quorum`|Configured quorum|Fixed-size clusters|
|`keep-oldest`|Oldest member|Cluster Singleton / leader-oriented workloads|
|`down-all`|No partition survives|Highly unstable networks|
|`lease-majority`|External distributed lease|Strong external coordination|

### Mental Model

```
                    Network Partition
                           │
             ┌─────────────┴─────────────┐
             │                           │
       Which side survives?        Should anything survive?
             │                           │
       ┌─────┼─────┐                     │
       │     │     │                     │
   Majority Quorum Oldest            Down All
       │     │     │
       └─────┴─────┘
             │
      External coordination?
             │
        Lease Majority
```

**Key idea:** SBR is fundamentally a **split-brain safety mechanism**. The strategy determines _which partition is allowed to continue_, while the tuning parameters determine _when Akka has enough confidence to make that decision_.
.