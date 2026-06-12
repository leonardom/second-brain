# Offline Sync in Mobile Apps

> Tags: #reliability #tradeoffs

Offline-first architecture ensures apps remain functional without connectivity, syncing state when the network is available.

---

## Why Offline-First Matters

- Mobile users frequently experience flaky connections (subway, elevators, rural areas)
- Background syncing improves perceived performance (optimistic UI)
- Legal/compliance requirements (medical, field service apps) may mandate local data

---

## Core Concepts

### Local-First Data

Store data locally as the primary source of truth. The server is a sync endpoint, not the sole source.

```
User Action → Local Store → UI Update (instant)
                  ↓
            Sync Queue → Server (eventually)
```

### Sync Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Full Sync** | Download entire dataset on reconnect | Small datasets, simple apps |
| **Delta Sync** | Transfer only changes since last sync | Large datasets, frequent changes |
| **Conflict-based** | Track changes per record with vector clocks/CRDTs | Collaborative apps |

---

## Conflict Resolution

When the same record is modified on multiple devices or offline, conflicts arise.

### Strategies

**Last Write Wins (LWW)**
- Simplest approach; use server or client timestamp
- Risk: silently discards changes; good only for non-critical data

**Server Wins**
- Server state always overrides client
- Good for: settings, configuration

**Client Wins**
- Client state always overrides server
- Good for: draft documents, local preferences

**Merge / Three-Way Merge**
- Compare base, client, and server versions; merge non-conflicting fields
- Conflicts on same field require user resolution

**CRDTs (Conflict-free Replicated Data Types)**
- Mathematically guaranteed convergence without coordination
- Types: G-Counter, PN-Counter, LWW-Register, OR-Set, RGA (text)
- Libraries: Automerge, Yjs

---

## Implementation Patterns

### Sync Queue

```
┌──────────────────────────────────────────────┐
│                  Device                       │
│                                              │
│  User Action → Optimistic Update → Local DB  │
│                       ↓                      │
│              Sync Queue (persisted)          │
│                       ↓                      │
│         NetworkMonitor: online?              │
│                       ↓ yes                  │
│          POST /sync (batch operations)       │
└──────────────────────────────────────────────┘
```

Each queue item:
```json
{
  "id": "uuid",
  "operation": "CREATE | UPDATE | DELETE",
  "entity": "Task",
  "payload": { "id": "t-1", "title": "Buy milk", "done": false },
  "timestamp": "2024-01-15T10:00:00Z",
  "retryCount": 0
}
```

### Idempotency

Every sync operation must be idempotent — replaying the same operation must produce the same result:

- Use UUIDs generated on the client (not server auto-increment IDs)
- Include the operation ID in the request; server deduplicates by ID

### Delta Sync with Watermarks

```
Client → GET /sync?since=2024-01-15T09:00:00Z
Server → { changes: [...], serverTime: "2024-01-15T10:30:00Z" }
Client → store serverTime as next watermark
```

- Server must store change history for at least the maximum expected offline period
- Soft-delete records (set `deleted_at`) so deletions propagate; never hard delete

---

## Platform-Specific Solutions

### iOS

- **Core Data + NSPersistentCloudKitContainer** — iCloud sync with conflict resolution
- **CloudKit** — Apple's backend for iOS sync (CKRecord, CKModifyRecordsOperation)
- **Realm** — local DB with optional Atlas Device Sync (MongoDB)

### Android

- **Room + WorkManager** — local SQLite via Room, deferred sync via WorkManager
- **Realm** — same cross-platform solution as iOS
- **Firebase Firestore** — offline-capable with automatic sync (limited query flexibility)

### Cross-Platform (React Native / Flutter)

- **WatermelonDB** — high-performance offline DB for React Native with sync protocol
- **PowerSync** — Postgres-backed sync service with SQLite on device
- **Amplify DataStore** — offline-first data layer backed by AppSync (GraphQL)
- **Electric SQL** — local-first Postgres sync (CRDT-based)

---

## Handling Network State

```typescript
// React Native example
import NetInfo from "@react-native-community/netinfo";

NetInfo.addEventListener(state => {
    if (state.isConnected && state.isInternetReachable) {
        syncQueue.flush(); // process pending operations
    }
});
```

Always distinguish:
- **Network connected** (to WiFi/cellular) ≠ **Internet reachable** (can reach your server)

---

## Testing Offline Scenarios

- Simulate offline: disable network in simulator/emulator or use Charles Proxy
- Test conflict resolution by modifying the same record from two devices
- Test large sync payloads (test with production-scale data volumes)
- Test interrupted sync (kill app mid-sync; verify resume on next open)
- Monitor battery impact of background sync (avoid excessive wakeups)

---

## Common Pitfalls

- **Auto-increment IDs** — clash when two offline clients create records; use UUID v4 or ULID
- **Sync on every keystroke** — debounce writes; batch changes before syncing
- **Ignoring clock skew** — device clocks drift; use server-assigned logical timestamps
- **No exponential backoff** — hammering a down server; implement backoff + jitter
- **Hard deleting records** — deletions won't propagate to other devices; use soft delete

---

## References

- [Local-First Software (Ink & Switch)](https://www.inkandswitch.com/local-first/)
- [Automerge](https://automerge.org/)
- [WatermelonDB Sync Protocol](https://watermelondb.dev/docs/Sync/Intro.html)
- [PowerSync](https://www.powersync.com/)
- [Electric SQL](https://electric-sql.com/)
