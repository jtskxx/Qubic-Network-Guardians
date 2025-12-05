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

<img width="745" height="565" alt="image" src="https://github.com/user-attachments/assets/aacaf0f8-b970-46e9-bc9e-17564d4dd78b" />


---

## How It Works

### 1. Set Up Your Node

Configure your bob or core-lite with:
- **Operator Seed** - Your unique identity (same format as Qubic wallet)
- **Node Name** - A display name for the leaderboard (optional, 3-20 characters)

```json
{
  "operator-seed": "yourseedhere...",
  "node-name": "CryptoKing"
}
```

### 2. Get Discovered

Your node is automatically discovered:

**Core-lite nodes:** Discovered through P2P network crawling (core-lite participates in peer exchange)

**Bob nodes:** Automatically announce themselves to the checker service on startup

### 3. Earn Points

The checker service monitors your node and awards points for:

| Metric | Weight | What It Measures |
|--------|--------|------------------|
| **Uptime** | 40% | Is your node responding? |
| **Sync Status** | 30% | Is your node up to date? |
| **Data Accuracy** | 20% | Does your node have correct data? |
| **Contribution** | 10% | Peer count, latency, features |

### 4. Climb the Leaderboard

<img width="900" height="480" alt="image" src="https://github.com/user-attachments/assets/fd980403-4085-4d6c-bf3d-a1222e02dc17" />

Total Guardians: 847 | Network Coverage: 23 countries
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

### Uptime (40% of score)

Your node is checked every 5 minutes. Each successful response = 1 point.

```
Maximum daily points: 288
Daily uptime score = (successful_checks / 288) × 100
```

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

Your node receives random data challenges daily. You must return correct hashes for historical ticks.

```
Daily challenges: 10
Points per correct answer: 10
Maximum daily points: 100
```

### Network Contribution (10% of score)

| Factor | Points |
|--------|--------|
| Accepts incoming connections | +20 |
| 5+ peer connections | +10 |
| 10+ peer connections | +20 |
| REST API available | +10 |
| Full history available | +15 |

---

## Your Node Profile

Each node gets a public profile page showing:

<img width="700" height="620" alt="image" src="https://github.com/user-attachments/assets/6f082f06-ecd8-41d7-822f-09724e93b3ba" />


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

To be eligible for epoch rewards:

1. Node online for at least **24 hours** during the epoch
2. Responded to at least **50%** of health checks
3. Passed at least **1 data challenge**

---

## Anti-Gaming Measures

We take fair play seriously:

| Measure | Description |
|---------|-------------|
| **Signed Responses** | Every response cryptographically signed with your seed |
| **IP Limits** | Maximum 3 nodes per IP address |
| **Random Challenges** | Data challenges use unpredictable historical ticks |
| **Cross-Verification** | Responses compared against known-good sources |

---

## Getting Started

### Step 1: Generate Your Seed

Use any Qubic wallet to generate a new seed, or create one specifically for your node.

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
A: Yes, but each needs a unique seed. Maximum 3 nodes per IP address.

**Q: What if my node goes offline?**
A: Your uptime score decreases, but there's no permanent penalty. Get back online and recover.

**Q: Can I change my node name?**
A: Yes, update your config and restart. The new name appears on the next check.

**Q: When are rewards paid?**
A: At the end of each epoch (Wednesday 12PM UTC), sent directly to your operator identity.

**Q: What's the reward pool size?**
A: Announced each epoch, depends on network growth and community allocation.

**Q: Is the code open source?**
A: Yes, both the node implementations and scoring algorithm are open source.

---

## Implementation Roadmap

### Phase 1: Identity & Discovery
- Add operator-seed and node-name configuration
- Implement node info protocol
- Build checker crawler
- Launch basic leaderboard

### Phase 2: Scoring System
- Implement uptime monitoring
- Add sync status tracking
- Build data challenge system
- Launch full leaderboard with profiles

### Phase 3: Rewards
- Epoch snapshot automation
- Reward distribution system
- Historical tracking
- Notification system

### Phase 4: Polish
- Mobile-friendly dashboard
- Geographic diversity map
- Performance optimizations
- Community features

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
    uint8_t  nodeType;           // 0=bob, 1=core-lite, 2=full-node
    uint8_t  version[3];         
    uint16_t epoch;              // Current epoch
    uint32_t tick;               // Current processed tick
    uint32_t latestNetworkTick;  // Latest tick seen on network
    uint32_t uptime;             // Seconds since node start
    uint16_t peerCount;          // Number of connected peers
    uint16_t capabilities;       // Feature flags (see below)

    // Identity
    uint8_t  operatorIdentity[32];  // Public key (m256i)
    char     nodeName[20];          // Display name (null-terminated)

    // Verification
    uint32_t challenge;          // Echo of request challenge
    uint64_t timestamp;          // Unix timestamp of response
    uint8_t  signature[64];      // Signature over response
};

// Capability flags
#define CAP_TCP_SERVER    0x0001  // Accepts TCP connections
#define CAP_REST_API      0x0002  // REST API available
#define CAP_FULL_HISTORY  0x0004  // Has historical tick data
#define CAP_LOG_EVENTS    0x0008  // Indexes logging events
#define CAP_ASSETS        0x0010  // Indexes asset data
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

#### REQUEST_DATA_CHALLENGE (Type 202)

```cpp
struct RequestDataChallenge {
    uint32_t tickNumber;     // Target tick
    uint8_t  challengeType;  // Type of data requested
    uint8_t  reserved[3];
    uint32_t nonce;          // Random nonce
};

// Challenge types
#define CHALLENGE_TICK_DIGEST     0  // K12 hash of TickData
#define CHALLENGE_TX_COUNT        1  // Number of transactions in tick
#define CHALLENGE_VOTE_DIGEST     2  // K12 hash of aggregated votes
```

#### RESPOND_DATA_CHALLENGE (Type 203)

```cpp
struct RespondDataChallenge {
    uint32_t tickNumber;
    uint8_t  challengeType;
    uint8_t  success;           // 1=have data, 0=don't have
    uint8_t  reserved[2];
    uint32_t nonce;             // Echo of request nonce
    uint8_t  responseData[32];  // Hash or count (zero-padded)
    uint8_t  signature[64];     // Signed response
};
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
  "latestNetworkTick": 18500002,
  "uptime": 86400,
  "peerCount": 4,
  "capabilities": ["tcp_server", "rest_api", "log_events"],
  "operatorIdentity": "ABCDEF...60chars",
  "challenge": "abc123...",
  "timestamp": 1699900000,
  "signature": "base64..."
}
```

#### POST /v1/dataChallenge

**Request:**
```json
{
  "tickNumber": 18400000,
  "challengeType": "tick_digest",
  "nonce": "random32bytes..."
}
```

**Response:**
```json
{
  "tickNumber": 18400000,
  "challengeType": "tick_digest",
  "success": true,
  "responseData": "abc123...32bytes",
  "nonce": "random32bytes...",
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
    "uptime": 18000,
    "sync": 13500,
    "accuracy": 9000,
    "contribution": 4723
  },
  "currentStatus": {
    "online": true,
    "tick": 18500000,
    "ticksBehind": 2,
    "peerCount": 6,
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

<img width="750" height="700" alt="image" src="https://github.com/user-attachments/assets/6850d633-6fde-4935-bb55-0d42f1ecc522" />


---

## D. Configuration Reference

### Bob Configuration (JSON)

```json
{
  "operator-seed": "55characterlowercaseseed...",
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
| `operator-seed` | Yes | 55-char lowercase seed for identity |
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
    uint64_t currentTime,
    uint32_t currentNetworkTick
) {
    // 1. Challenge must match
    if (resp.challenge != req.challenge)
        return false;

    // 2. Timestamp must be recent (within 60 seconds)
    if (abs((int64_t)currentTime - (int64_t)resp.timestamp) > 60)
        return false;

    // 3. Tick must be plausible
    if (resp.tick > currentNetworkTick + 10)
        return false;

    // 4. Verify signature
    uint8_t digest[32];
    KangarooTwelve(
        &resp,
        offsetof(RespondNodeInfo, signature),
        digest,
        32
    );

    if (!verify(resp.operatorIdentity, digest, resp.signature))
        return false;

    // 5. Validate node name (if present)
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
