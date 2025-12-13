# Qubic Network Guardians

A gamification system to incentivize running Qubic lite nodes and strengthen the network.

---

## The Problem

The Qubic network relies on high-spec baremetal nodes (2TB+ RAM) which are expensive to run. To make the network more accessible and resilient, we created lighter alternatives:

- **qubic-bob** - Lite node/indexer (16GB RAM)
- **qubic-core-lite** - Full node running on OS (64GB RAM)

But how do we encourage people to run these nodes?

---

## The Solution

Qubic Network Guardians rewards node operators for keeping their nodes online, synced, and serving accurate data.

<img width="745" height="565" alt="image" src="https://github.com/user-attachments/assets/a2a57dcd-c6e3-43b7-b91a-e59a646259d6" />


---

## How It Works

### 1. Set Up Your Node

Configure your bob or core-lite with:
- **Operator Seed** - Your unique identity (55 lowercase letters, same as Qubic wallet seed)
- **Node Name** - A display name for the leaderboard (optional, 3-20 characters)

```json
{
  "operator-seed": "yourseedhere...",
  "node-name": "CryptoKing"
}
```

> **Note:** Config field names are preliminary and may change to align with codebase during implementation.

### 2. Get Discovered

Your node is automatically discovered:

**Core-lite nodes:** Discovered through P2P network crawling (core-lite participates in peer exchange)

**Bob nodes:** Automatically announce themselves to the checker service on startup

### 3. Earn Points

The checker service monitors your node and awards points for:

| Metric | Weight | What It Measures |
|--------|--------|------------------|
| **Uptime** | 50% | Is your node responding? |
| **Sync Status** | 30% | Is your node up to date? |
| **Data Accuracy** | 20% | Does your node return correct data? |

### 4. Climb the Leaderboard

<img width="900" height="480" alt="image" src="https://github.com/user-attachments/assets/389b2f58-7094-40d1-98d9-a6109a27e03e" />

### 5. Earn Rewards

QU rewards distributed proportionally based on your score each epoch:

```
Your Reward = (Your Score / Total Network Score) × Epoch Pool
```

**Example:**
```
Epoch Pool:          1,000,000,000 QU
Total Network Score:     5,000,000 points
Your Score:                 50,000 points

Your Reward = (50,000 / 5,000,000) × 1,000,000,000 = 10,000,000 QU
```

The more you contribute, the more you earn. Simple and fair.

Rewards are sent directly to your operator identity address at the end of each epoch (Wednesday 12PM UTC).

---

## Node Requirements

### Bob (Lite Indexer)

| Requirement | Minimum |
|-------------|---------|
| RAM | 16 GB |
| CPU | 4 cores (AVX2) |
| Network | Stable connection |

**To participate:**
- Set `operator-seed` in config
- Enable `run-server: true`
- Node will announce itself to the checker service automatically

> **Future:** Bob may add full P2P capability later, enabling discovery through peer exchange like core-lite.

### Core-Lite (Full Node on OS)

| Requirement | Minimum |
|-------------|---------|
| RAM | 64 GB |
| CPU | 8 cores (AVX2/AVX512) |
| Network | 1 Gbps recommended |

**To participate:**
- Set `operatorSeed` in private_settings.h
- Node is automatically discoverable via P2P

---

## Scoring Details

### Uptime (50% of score)

Your node is checked at randomized intervals.

```
Daily uptime score = (successful_checks / expected_checks) × 100
```

**Distributed Monitoring:**
- Monitor agents run across **multiple datacenters** worldwide
- If one agent can't reach your node, another agent retries from a different location
- Prevents false negatives from regional network issues
- Check timing is randomized

### Sync Status (30% of score)

How close is your node to the current network tick?

| Ticks Behind | Score |
|--------------|-------|
| 0-10 | 100% |
| 11-50 | 80% |
| 51-150 | 60% |
| 151-350 | 40% |
| 351-676 | 20% |
| 676+ | 5% |

### Data Accuracy (20% of score)

Verifies that your node reports correct blockchain state.

**How it works:**
1. During each uptime check, the checker requests `RespondNodeInfo` from your node
2. Your node responds with its current `tick` and `epoch`
3. Checker compares these values against trusted sources (computors, verified nodes)
4. If values match within tolerance → accurate

**What's checked (both bob and core-lite):**

| Field | Tolerance | Description |
|-------|-----------|-------------|
| `epoch` | Exact match | Must match current network epoch |
| `tick` | ±10 ticks | Your processed tick vs network tick |

```
Check frequency: Once per uptime check
Score: 100% if accurate, 0% if mismatch
```

---

## Your Node Profile

Each node gets a public profile page showing:

<img width="700" height="620" alt="image" src="https://github.com/user-attachments/assets/9fe066f8-56aa-4fd7-a0c1-c46231121a4b" />

---

## Network Statistics

The dashboard also shows overall network health:

- Total active guardians
- Geographic distribution map
- Average network uptime
- Total rewards distributed
- New guardians this epoch

---

## Minimum Requirements for Rewards

To be eligible for epoch rewards, your node must meet **all** thresholds:

| Requirement | Threshold |
|-------------|-----------|
| Node uptime | ≥ 70% of epoch |
| Data accuracy rate | ≥ 80% |

Nodes failing any threshold receive **zero rewards** for that epoch, regardless of their score.

---

## Smart Contract & Funding

| Phase | Approach |
|-------|----------|
| **Short-term Launch** | Go live without smart contract first. Initial reward pot funded by donations and sponsorships. |
| **Long-term Goal** | Smart contract holds reward pot, funded by computors (similar to Qearn/CCF model). SC retrieves node stats from Network Guardians service via Operator Message (OM). |

---

## Anti-Gaming Measures

We take fair play seriously:

| Measure | Description |
|---------|-------------|
| **Signed Responses** | Every response cryptographically signed with your seed |
| **One IP = One Node** | Only one node per IP address is eligible for rewards |
| **Cross-Verification** | Data responses compared against known-good sources |
| **Relay Detection** | Cryptographic identity binding prevents proxy/relay nodes (see below) |

### Relay/Proxy Node Detection

A relay node is a lightweight server that forwards requests to a real node without doing actual work. We prevent this using cryptographic identity binding:

**Cryptographic Identity Binding**
- Every `RespondNodeInfo` is signed with the operator's private key (FourQ/SchnorrQ)
- Signature computed: `sign(privateKey, K12(response_data))`
- Response includes `challenge` (echoed from request) + `timestamp` (milliseconds)
- Relay cannot sign without the operator's seed
- If relay forwards to real node, the signature is from the real node's identity — not the relay's claimed identity

**Why Relays Fail:**
| Attack | Why It Fails |
|--------|--------------|
| Forward requests to real node | Signature belongs to real node's identity, not relay's claimed identity |
| Cache responses | Challenge mismatch (unique per request) |

**Penalties:**
- First detection: Warning, node flagged for increased monitoring
- Confirmed relay: Node blacklisted, operator identity banned for current epoch
- Repeat offenders: Permanent ban from rewards program

---

## Getting Started

### Step 1: Generate Your Seed

Use any Qubic wallet to generate a new seed, or create one specifically for your node. You can create a wallet at [wallet.qubic.org](https://wallet.qubic.org).

> **Important:** Keep your seed safe! It's both your identity and your reward address.

### Step 2: Configure Your Node

**For bob:**
```json
{
  "operator-seed": "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabc",
  "node-name": "MyAwesomeNode",
  "run-server": true,
  "server-port": 21841
}
```

**For core-lite** (in `private_settings.h`):
```cpp
static unsigned char operatorSeed[55 + 1] =
    "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabc";
static char nodeName[21] = "MyAwesomeNode";
```

### Step 3: Start Your Node

Run your node normally. The checker will discover it and you'll appear on the leaderboard.

### Step 4: Monitor Your Progress

Visit `guardians.qubic.org/node/{your-identity}` to see your stats.

---

## Frequently Asked Questions

**Q: Do I need to register my node?**
A: No. Core-lite nodes are discovered through P2P crawling. Bob nodes automatically announce themselves to the checker on startup.

**Q: Can I run multiple nodes?**
A: Only one node per IP address is eligible for rewards. If you have multiple IPs (e.g., different servers), each can run one eligible node with a unique seed.

**Q: What if my node goes offline?**
A: Your uptime score decreases, but there's no permanent penalty. Get back online and recover.

**Q: Can I change my node name?**
A: Yes, update your config and restart. The new name appears on the next check.

**Q: When are rewards paid?**
A: At the end of each epoch (Wednesday 12PM UTC), sent directly to your operator identity.

**Q: What's the reward pool size?**
A: Announced each epoch. Initially funded by donations/sponsorships. Long-term goal is a smart contract funded by computors (similar to Qearn/CCF).

**Q: Is the code open source?**
A: Yes, both the node implementations and scoring algorithm are open source.

---

## Links

- [qubic-bob Repository](https://github.com/krypdkat/qubicbob)
- [qubic-core-lite Repository](https://github.com/hackerby888/qubic-core-lite)
- [Guardian Leaderboard](https://guardians.qubic.org) *(coming soon)*
- [Qubic Documentation](https://docs.qubic.org)

---

---

# Technical Appendix

> **Note:** This is a draft. The following sections contain preliminary technical ideas and are subject to change during implementation.

---

## A. Protocol Extensions

### New Message Types

#### REQUEST_NODE_INFO (Type 200)

```cpp
struct RequestNodeInfo {
    uint32_t challenge;      // Random nonce for replay protection
    uint8_t  reserved[4];    // Future use
};
```

#### RESPOND_NODE_INFO (Type 201)

```cpp
struct RespondNodeInfo {
    // Node metadata
    uint8_t  nodeType;           // 0=bob, 1=core-lite
    uint8_t  version[3];
    uint16_t epoch;              // Current epoch
    uint32_t tick;               // Current processed tick

    // Identity
    uint8_t  operatorIdentity[32];  // Public key (m256i)
    char     nodeName[20];          // Display name (null-terminated)

    // Verification
    uint32_t challenge;          // Echo of request challenge
    uint64_t timestamp;          // Unix timestamp (milliseconds) of response
    uint8_t  signature[64];      // FourQ/SchnorrQ signature over all above fields
};
```

**Signature computation:**
```cpp
uint8_t digest[32];
KangarooTwelve(
    &response,                    // Start of struct
    offsetof(RespondNodeInfo, signature),  // Everything before signature
    digest,
    32
);
sign(operatorPrivateKey, digest, response.signature);
```

---

## B. REST API Endpoints

### Node Endpoints (Bob)

#### GET /v1/nodeInfo

Returns signed node information.

**Query Parameters:**
- `challenge` (optional): 32-byte hex challenge

**Response:**
```json
{
  "nodeType": "bob",
  "nodeName": "CryptoKing",
  "version": "1.2.3",
  "epoch": 140,
  "tick": 18500000,
  "operatorIdentity": "ABCDEF...60chars",
  "challenge": "abc123...",
  "timestamp": 1699900000000,
  "signature": "base64..."
}
```

### Leaderboard API (guardians.qubic.org)

#### GET /api/v1/leaderboard

**Query Parameters:**
- `limit` (default: 100, max: 500)
- `offset` (default: 0)

**Response:**
```json
{
  "totalNodes": 847,
  "activeNodes": 812,
  "lastUpdated": "2024-01-15T12:00:00Z",
  "entries": [
    {
      "rank": 1,
      "nodeName": "CryptoKing",
      "identity": "ABCDEF...WXYZ",
      "nodeType": "bob",
      "score": 98547,
      "uptimePercent": 99.8,
      "lastSeen": "2024-01-15T11:59:00Z",
      "country": "DE"
    }
  ]
}
```

#### GET /api/v1/node/{identity}

**Response:**
```json
{
  "identity": "ABCDEF...",
  "nodeName": "CryptoKing",
  "nodeType": "bob",
  "version": "1.2.3",
  "rank": 127,
  "totalScore": 45223,
  "breakdown": {
    "uptime": 22612,
    "sync": 13567,
    "accuracy": 9044
  },
  "currentStatus": {
    "online": true,
    "tick": 18500000,
    "ticksBehind": 2,
    "lastCheck": "2024-01-15T12:00:00Z"
  },
  "history": [
    {"date": "2024-01-14", "score": 287, "uptime": 98.5},
    {"date": "2024-01-13", "score": 291, "uptime": 100.0}
  ],
  "rewards": {
    "pending": 10000000,
    "totalEarned": 58000000,
    "lastPayout": "2024-01-14T00:00:00Z"
  }
}
```

#### GET /api/v1/stats

**Response:**
```json
{
  "totalNodes": 847,
  "activeNodes": 812,
  "nodesByType": {
    "bob": 623,
    "core-lite": 189
  },
  "nodesByCountry": {
    "US": 234,
    "DE": 156,
    "FR": 89
  },
  "averageUptime": 94.5,
  "averageSyncPercent": 97.2,
  "epochRewardPool": 1000000000,
  "totalRewardsDistributed": 15000000000
}
```

---

## C. Checker Service Architecture

<img width="850" height="780" alt="image" src="https://github.com/user-attachments/assets/f1ec9d1a-b229-41aa-847d-054da0510f47" />


### Agent Synchronization

**Coordinator responsibilities:**
- Assigns nodes to agents (load balancing)
- Randomizes check timing per node (not predictable)
- Handles retry logic: if Agent A fails to reach node, Agent B retries
- Aggregates results from multiple agents
- Resolves conflicts (e.g., Agent A says online, Agent B says offline → majority wins)

**Retry policy:**
```
1. Agent A checks node → timeout/failure
2. Coordinator assigns retry to Agent B (different datacenter)
3. Agent B checks node → success/failure
4. If still failing, Agent C retries (third datacenter)
5. After 3 failed attempts from different locations → mark as offline
```

**Scoring:** If any attempt succeeds, the check passes (no penalty). Only 3 consecutive failures from different locations counts as failed check. This prevents penalizing nodes for regional network issues outside their control.

**Consistency:**
- All agents sync to same coordinator database
- Check assignments use distributed locking to prevent duplicates
- Results timestamped with agent ID for audit trail

### Database Schema (v1)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              ERM DIAGRAM                                        │
└─────────────────────────────────────────────────────────────────────────────────┘

┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│    operators     │       │      nodes       │       │     agents       │
├──────────────────┤       ├──────────────────┤       ├──────────────────┤
│ *operator_id PK  │◄──────│  operator_id FK  │       │ *agent_id PK     │
│  identity UQ     │       │ *node_id PK      │       │  name            │
│  public_key      │       │  ip_address UQ   │       │  datacenter      │
│  created_at      │       │  node_type       │       │  region          │
│  is_banned       │       │  node_name       │       │  is_active       │
│  ban_reason      │       │  version         │       │  last_heartbeat  │
│  ban_expires     │       │  first_seen      │       └────────┬─────────┘
└──────────────────┘       │  last_seen       │                │
                           │  is_active       │                │
                           │  is_banned       │                │
┌──────────────────┐       └────────┬─────────┘                │
│      epochs      │                │                          │
├──────────────────┤                │                          │
│ *epoch_number PK │       ┌────────┴────────┐                 │
│  start_tick      │       │                 │                 │
│  end_tick        │       ▼                 ▼                 │
│  start_time      │  ┌───────────────┐ ┌───────────────┐      │
│  end_time        │  │ uptime_checks │ │  sync_checks  │      │
│  reward_pool     │  ├───────────────┤ ├───────────────┤      │
│  status          │  │ *check_id PK  │ │ *check_id PK  │      │
└────────┬─────────┘  │  node_id FK   │ │  node_id FK   │      │
         │            │  agent_id FK ─┼─┼► agent_id FK ─┼──────┘
         │            │  epoch FK ◄───┼─┼─ epoch FK     │
         │            │  timestamp    │ │  timestamp    │
         │            │  success      │ │  current_tick │
         │            │  response_ms  │ │  network_tick │
         │            │  retry_count  │ │  ticks_behind │
         │            │  data_accurate│ │  sync_score   │
         │            │  error_msg    │ └───────────────┘
         │            └───────────────┘
         │
         ▼
┌──────────────────┐       ┌──────────────────┐
│  epoch_scores    │       │     rewards      │
├──────────────────┤       ├──────────────────┤
│ *score_id PK     │       │ *reward_id PK    │
│  node_id FK      │       │  node_id FK      │
│  epoch FK        │       │  epoch FK        │
│  uptime_score    │       │  score_id FK     │◄───┐
│  sync_score      │       │  amount_qu       │    │
│  accuracy_score  │       │  share_percent   │    │
│  total_score     │───────│  tx_hash         │    │
│  rank            │       │  status          │    │
│  meets_uptime    │       │  paid_at         │    │
│  meets_accuracy  │───────┴──────────────────┘    │
│  calculated_at   │                               │
└──────────────────┴───────────────────────────────┘
```

**Table Definitions:**

```sql
-- Epochs: Track each epoch period
CREATE TABLE epochs (
    epoch_number    INT PRIMARY KEY,
    start_tick      BIGINT NOT NULL,
    end_tick        BIGINT,
    start_time      TIMESTAMP NOT NULL,
    end_time        TIMESTAMP,
    reward_pool     BIGINT DEFAULT 0,        -- Total QU for this epoch
    status          VARCHAR(20) DEFAULT 'active'  -- active, completed, paying, paid
);

-- Operators: Node operators identified by their Qubic identity
CREATE TABLE operators (
    operator_id     SERIAL PRIMARY KEY,
    identity        CHAR(60) UNIQUE NOT NULL, -- Qubic identity (60 chars)
    public_key      BYTEA NOT NULL,           -- 32-byte public key
    created_at      TIMESTAMP DEFAULT NOW(),
    is_banned       BOOLEAN DEFAULT FALSE,
    ban_reason      TEXT,
    ban_expires     TIMESTAMP
);

-- Nodes: Individual nodes being monitored
CREATE TABLE nodes (
    node_id         SERIAL PRIMARY KEY,
    operator_id     INT REFERENCES operators(operator_id),
    ip_address      INET UNIQUE NOT NULL,     -- One IP = One Node
    node_type       VARCHAR(20) NOT NULL,     -- 'bob', 'core-lite'
    node_name       VARCHAR(20),
    version         VARCHAR(20),
    first_seen      TIMESTAMP DEFAULT NOW(),
    last_seen       TIMESTAMP,
    is_active       BOOLEAN DEFAULT TRUE,
    is_banned       BOOLEAN DEFAULT FALSE
);

-- Agents: Distributed monitoring agents
CREATE TABLE agents (
    agent_id        SERIAL PRIMARY KEY,
    name            VARCHAR(50) NOT NULL,
    datacenter      VARCHAR(50) NOT NULL,
    region          VARCHAR(20) NOT NULL,     -- 'US', 'EU', 'ASIA', etc.
    is_active       BOOLEAN DEFAULT TRUE,
    last_heartbeat  TIMESTAMP
);

-- Uptime Checks: Record of each uptime check (includes data accuracy)
CREATE TABLE uptime_checks (
    check_id        BIGSERIAL PRIMARY KEY,
    node_id         INT REFERENCES nodes(node_id),
    agent_id        INT REFERENCES agents(agent_id),
    epoch           INT REFERENCES epochs(epoch_number),
    timestamp       TIMESTAMP NOT NULL,
    success         BOOLEAN NOT NULL,
    response_ms     INT,                      -- Response time in ms
    retry_count     SMALLINT DEFAULT 0,       -- How many retries needed
    data_accurate   BOOLEAN,                  -- Did returned data match known-good sources?
    error_msg       TEXT                      -- Error message if failed
);

-- Sync Checks: Node synchronization status
CREATE TABLE sync_checks (
    check_id        BIGSERIAL PRIMARY KEY,
    node_id         INT REFERENCES nodes(node_id),
    agent_id        INT REFERENCES agents(agent_id),
    epoch           INT REFERENCES epochs(epoch_number),
    timestamp       TIMESTAMP NOT NULL,
    current_tick    BIGINT NOT NULL,          -- Node's current tick
    network_tick    BIGINT NOT NULL,          -- Network's current tick
    ticks_behind    INT NOT NULL,             -- Difference
    sync_score      SMALLINT NOT NULL         -- Score based on ticks behind (0-100)
);

-- Epoch Scores: Aggregated scores per node per epoch
CREATE TABLE epoch_scores (
    score_id        BIGSERIAL PRIMARY KEY,
    node_id         INT REFERENCES nodes(node_id),
    epoch           INT REFERENCES epochs(epoch_number),
    uptime_score    INT NOT NULL,             -- Points from uptime checks
    sync_score      INT NOT NULL,             -- Points from sync status
    accuracy_score  INT NOT NULL,             -- Points from data accuracy checks
    total_score     INT NOT NULL,             -- Weighted total
    rank            INT,                      -- Rank in this epoch
    meets_uptime    BOOLEAN NOT NULL,         -- Met ≥70% uptime requirement
    meets_accuracy  BOOLEAN NOT NULL,         -- Met ≥80% accuracy requirement
    calculated_at   TIMESTAMP DEFAULT NOW(),
    UNIQUE(node_id, epoch)
);

-- Rewards: Payout records
CREATE TABLE rewards (
    reward_id       BIGSERIAL PRIMARY KEY,
    node_id         INT REFERENCES nodes(node_id),
    epoch           INT REFERENCES epochs(epoch_number),
    score_id        BIGINT REFERENCES epoch_scores(score_id),
    amount_qu       BIGINT NOT NULL,          -- Reward amount in QU
    share_percent   DECIMAL(10,6),            -- Percentage of pool
    tx_hash         CHAR(64),                 -- Transaction hash when paid
    status          VARCHAR(20) DEFAULT 'pending', -- pending, paid, failed
    paid_at         TIMESTAMP,
    UNIQUE(node_id, epoch)
);

-- Indexes for common queries
CREATE INDEX idx_uptime_node_epoch ON uptime_checks(node_id, epoch);
CREATE INDEX idx_uptime_timestamp ON uptime_checks(timestamp);
CREATE INDEX idx_sync_node_epoch ON sync_checks(node_id, epoch);
CREATE INDEX idx_scores_epoch_rank ON epoch_scores(epoch, rank);
CREATE INDEX idx_nodes_ip ON nodes(ip_address);
CREATE INDEX idx_operators_identity ON operators(identity);
```

---

## D. Configuration Reference

### Bob Configuration (JSON)

```json
{
  "operator-seed": "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabc",
  "node-name": "MyNodeName",
  "run-server": true,
  "server-port": 21841,

  "trusted-node": ["BM:1.2.3.4:21841"],
  "keydb-url": "tcp://127.0.0.1:6379",
  "log-level": "info"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `operator-seed` | Yes | 55 lowercase letters for identity |
| `node-name` | No | Display name (3-20 chars, alphanumeric + underscore) |
| `run-server` | Yes | Must be `true` so checker can connect back |
| `server-port` | No | TCP port (default: 21841) |

On startup, bob automatically announces itself to the Guardian checker service. The checker then connects back to verify identity and begin monitoring.

### Core-Lite Configuration (C++)

In `private_settings.h`:

```cpp
// Guardian identity
static unsigned char operatorSeed[55 + 1] =
    "yourseedhere...";

// Display name (max 20 chars)
static char nodeName[21] = "MyCoreLiteNode";
```

---

## E. Response Validation

```cpp
bool validateNodeInfoResponse(
    const RequestNodeInfo& req,
    const RespondNodeInfo& resp,
    uint64_t currentTimeMs,
    uint32_t currentNetworkTick,
    uint16_t currentEpoch
) {
    // 1. Challenge must match (replay protection)
    if (resp.challenge != req.challenge)
        return false;

    // 2. Timestamp must be recent (replay protection)
    if (abs((int64_t)currentTimeMs - (int64_t)resp.timestamp) > 60000)
        return false;

    // 3. Epoch must match current network epoch
    if (resp.epoch != currentEpoch)
        return false;

    // 4. Tick must be plausible (within tolerance of network tick)
    if (resp.tick > currentNetworkTick + 10)
        return false;

    // 5. Verify FourQ signature
    uint8_t digest[32];
    KangarooTwelve(
        &resp,
        offsetof(RespondNodeInfo, signature),
        digest,
        32
    );
    if (!verify(resp.operatorIdentity, digest, resp.signature))
        return false;

    // 6. Validate node name (if present)
    if (resp.nodeName[0] != '\0') {
        size_t len = strnlen(resp.nodeName, 20);
        if (len < 3) return false;
        for (size_t i = 0; i < len; i++) {
            char c = resp.nodeName[i];
            if (!isalnum(c) && c != '_') return false;
        }
    }

    return true;
}
```

---

## License

The Anti-Military License. See [LICENSE.md](LICENSE.md).

This project is licensed under the Anti-Military License. However, it includes
some code licensed under the MIT License. The MIT licensed code is used with
permission and retains its original MIT license.

The MIT licensed code is from the following sources:
- [uint128_t](https://github.com/calccrypto/uint128_t)

