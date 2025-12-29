# Nostr Relay Specifications

## Relay Rankings (2025/11/10)

Source: [nostr.watch](https://nostr.watch/relays/software)

| Name | Relays | Versions | Share | Based-on |
|------|--------|----------|-------|----------|
| hoytech/strfry | 237 | 35 | 26.9% | - |
| ~gheartsfield/nostr-rs-relay | 205 | 10 | 23.3% | - |
| bitvora/haven | 71 | 15 | 8.1% | khatru |
| cameri/nostream | 55 | 8 | 6.3% | - |
| bitvora/wot-relay | 25 | 5 | 2.8% | khatru |
| fiatjaf/khatru | 18 | 3 | 2.0% | - |

## Limit Parameter Behavior

This document compares the behavior of the `limit` filter parameter across different Nostr relay implementations.

### Summary Table

| Relay Implementation | Default Max Limit | Config Parameter | Behavior |
|---------------------|-------------------|------------------|----------|
| strfry | [500](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91) | `relay.maxFilterLimit` | Returns min(client_limit, 500) events per filter |
| nostream | [5000](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L162) | `limits.client.subscription.maxLimit` | Returns min(client_limit, 5000) events per subscription |
| nostr-rs-relay | No explicit limit | Not found in default config | Returns all matching events when limit not specified |
| khatru (framework) | No default limit | Not applicable | Implementation-dependent, no built-in max limit |
| haven (khatru-based) | No limit | Not configured | Returns all matching events via eventstore |
| wot-relay (khatru-based) | No limit | Not configured | Returns all matching events via eventstore |

## Rate Limiting

This table shows the rate limits each relay enforces on client requests.

| Relay | Type | Max Subscriptions | Event Submission Rate | Filter/REQ Rate | Connection Rate | Notes |
|-------|------|-------------------|----------------------|-----------------|-----------------|-------|
| strfry | - | [20](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L89) | Not configured by default | Not configured | Not configured | All limits are configurable |
| nostream | Kind [0](https://github.com/nostr-protocol/nips/blob/master/01.md),[3](https://github.com/nostr-protocol/nips/blob/master/02.md),[40](https://github.com/nostr-protocol/nips/blob/master/28.md),[41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [6 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L108-L115) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Metadata, contacts, channel events |
| nostream | Kind [1](https://github.com/nostr-protocol/nips/blob/master/01.md),[2](https://github.com/nostr-protocol/nips/blob/master/01.md),[4](https://github.com/nostr-protocol/nips/blob/master/04.md),[42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [12 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L116-L123) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Notes, DMs, channel messages |
| nostream | Kind [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md),[43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [30 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L124-L131) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Deletions, reactions, channel events |
| nostream | Kind [10000-19999,30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [24 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L132-L141) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Replaceable events |
| nostream | Kind [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [60 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L142-L147) | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Ephemeral events |
| nostream | All events | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L110) | [720 events/hour](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L148-L150) overall limit | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) | Combined across all kinds |
| nostr-rs-relay | - | No limit | Configurable (default: unlimited) | Configurable (default: unlimited) | Not configured | Optional: [messages_per_sec](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L115), [subscriptions_per_min](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L121) |
| khatru | - | No limit | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L12) (max 10 tokens) | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L17) (max 100 tokens) | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L21) (max 100 tokens) | Framework. Via [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |
| haven | Private | No limit | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L61) (max 100 tokens) | Not separately configured | [3 conn/5min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L66) | Khatru-based |
| haven | Chat | No limit | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L72) (max 100 tokens) | Not separately configured | [3 conn/3min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L77) | Khatru-based |
| haven | Inbox | No limit | [10 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L83) (max 20 tokens) | Not separately configured | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L88) | Khatru-based |
| haven | Outbox | No limit | [10 events/60min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L94) (max 100 tokens) | Not separately configured | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L99) | Khatru-based |
| wot-relay | - | No limit | [5 events/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L131) (max 30 tokens) | [5 filters/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L137) (max 30 tokens) | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L141) (max 30 tokens) | Khatru-based. Stricter than khatru defaults |

**Important notes:**
- **Max Subscriptions**: Maximum concurrent REQ subscriptions per connection. "No limit" means the relay doesn't enforce a cap (framework-dependent)
- **Token bucket algorithm**: Most relays use token bucket rate limiting where tokens refill over time
- **Rate limits ≠ Result limits**: Rate limits control request frequency, not the number of events returned per request
- **Per-IP vs Per-Connection**: Most implementations apply limits per IP address

---

## Time-based Restrictions

### Event Submission Time Validation

This table shows how relays validate the `created_at` timestamp when clients submit new events.

| Relay | Max Future Offset | Max Past Offset | Notes |
|-------|------------------|-----------------|-------|
| strfry | [+900s (15 min)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L24) | [-94,608,000s (~3 years)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L27) | Rejects events outside this range |
| nostream | [+900s (15 min)](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L90) | No limit | Only future events rejected |
| nostr-rs-relay | [+1,800s (30 min)](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L105) | No limit | Only future events rejected |
| khatru | Not enforced | Not enforced | Framework doesn't enforce by default |
| haven | Not enforced | Not enforced | Inherits khatru behavior |
| wot-relay | Not enforced | Not enforced | Inherits khatru behavior |

**Purpose of time validation:**
- **Future offset limit**: Prevents clients from creating events with timestamps too far in the future
- **Past offset limit**: Prevents backdate attacks where malicious users create events with very old timestamps

### Event Storage/Deletion Policies

This table shows how relays manage stored events over time.

| Relay | Ephemeral Event Age | Ephemeral Event Lifetime | Regular Event Max Age | Notes |
|-------|---------------------|--------------------------|----------------------|-------|
| strfry | Reject if [>60s](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L30) old | Auto-delete after [300s](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L33) | - | Kind 20000-29999 only |
| nostream | - | - | - | No automatic deletion |
| nostr-rs-relay | - | - | - | No automatic deletion |
| khatru | - | - | - | Implementation-dependent |
| haven | - | - | - | No automatic deletion |
| wot-relay | - | - | Delete after [365 days](https://github.com/bitvora/wot-relay/blob/24b51de9/.env.example#L27) | Configurable via MAX_AGE_DAYS |

**Key differences:**
- **Ephemeral events** (kind 20000-29999): Temporary events not expected to be stored long-term
- **Regular events**: Standard events that relays typically store indefinitely
- **Auto-deletion**: Some relays automatically remove old events to manage storage

---


## Detailed Analysis

### 1. strfry (C++)

**Configuration File:** `strfry.conf`

**Default Setting:**
```conf
relay {
    # Maximum records that can be returned per filter
    maxFilterLimit = 500
}
```

**Behavior:**
- When a client requests events with `limit: N`, strfry returns **min(N, 500)** events per filter
- If client doesn't specify limit or specifies limit > 500, strfry caps it at 500
- This is a per-filter limit, not per-subscription limit

**Size Limits:**
- Event size: 65,536 bytes (64 KB) - normalized JSON
- WebSocket payload: 131,072 bytes (128 KB)
- Tag value size: 1,024 bytes
- Max filters per REQ: 200

**Time Limits:**
- Reject events newer than 900 seconds (15 minutes) in the future
- Reject events older than 94,608,000 seconds (~3 years)
- Reject ephemeral events older than 60 seconds
- Ephemeral events lifetime: 300 seconds (5 minutes)

**NIPs Supported:** 1, 2, 4, 9, 11, 22, 28, 40, 70, 77

---

### 2. nostream (TypeScript)

**Configuration File:** `resources/default-settings.yaml`

**Default Setting:**
```yaml
limits:
  client:
    subscription:
      maxSubscriptions: 10
      maxFilters: 10
      maxFilterValues: 2500
      maxSubscriptionIdLength: 256
      maxLimit: 5000
      minPrefixLength: 4
```

**Behavior:**
- When a client requests events with `limit: N`, nostream returns **min(N, 5000)** events
- The `maxLimit: 5000` setting controls the maximum allowed limit value
- Also enforces:
  - Maximum 10 concurrent subscriptions per client
  - Maximum 10 filters per subscription
  - Maximum 2500 filter values total

**Size Limits:**
- Network payload: 524,288 bytes (512 KB)
- Event content: 102,400 bytes (100 KB) for kind ranges 0-10, 40-49, and 11-39, 50-max

**Time Limits:**
- Event created_at: maximum 900 seconds (15 minutes) in the future
- Event created_at: maximum 0 seconds in the past (no past events rejected by time)

**Rate Limits (per event kind):**
- Kind 0, 3, 40, 41: 6 events/minute
- Kind 1, 2, 4, 42: 12 events/minute
- Kind 5-7, 43-49: 30 events/minute
- Kind 10000-19999, 30000-39999: 24 events/minute (replaceable)
- Kind 20000-29999: 60 events/minute (ephemeral)
- All events: 720 events/hour

**NIPs Supported:** 01, 02, 04, 09, 11, 11a, 12, 13, 15, 16, 20, 22, 28, 33, 40

**Additional Features:**
- Payment processor integration (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL)
- Extensive rate limiting per event kind

---

### 3. nostr-rs-relay (Rust)

**Configuration File:** `config.toml`

**Default Setting:**
- No explicit `maxLimit` or similar parameter found in default configuration
- Configuration focuses on rate limiting, connection limits, and event size limits

**Behavior:**
- When `limit` is specified in filter: applies SQL LIMIT clause with that value
- When `limit` is NOT specified: no LIMIT clause added to SQL query
- **Returns all matching events when limit not specified** (potentially unlimited)

**Source Code Evidence:**
```rust
// src/repo/sqlite.rs:1093-1094
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

**Size Limits (default values):**
- Event size: 131,072 bytes (128 KB)
- WebSocket message: 131,072 bytes (128 KB)
- WebSocket frame: 131,072 bytes (128 KB)

**Time Limits:**
- Reject events more than 1,800 seconds (30 minutes) in the future

**Rate Limits (configurable):**
- Messages per second: configurable (default: unlimited)
- Subscriptions per minute: configurable (default: unlimited)

**NIPs Supported:** 01, 02, 05, 09, 11, 12, 15, 16, 20, 22, 28, 33, 40, 42

**Notable Configuration Options:**
```toml
[options]
reject_future_seconds = 1800

[limits]
# messages_per_sec = 5
# subscriptions_per_min = 0
# max_event_bytes = 131072
# max_ws_message_bytes = 131072
# max_ws_frame_bytes = 131072
```

---

### 4. khatru (Go Framework)

**Configuration:** Code-based, no config file

**Default Setting:**
- No built-in maximum limit enforcement in the framework
- Developers must implement their own limit policies

**Behavior:**
- Framework handles `LimitZero` flag to skip queries with `limit: 0`
- Actual limit enforcement depends on the relay implementation using Khatru
- Provides rate limiting helpers but not result limit caps

**Framework Features:**
- Custom event/filter acceptance policies
- Custom AUTH handlers
- Pluggable storage backends
- Built-in policy helpers in `policies` package

**Default Size Limits:**
- Max message size: 512,000 bytes (500 KB)

**Default Rate Limits (via `ApplySaneDefaults`):**
- Event rate: 2 events per 3 minutes (max 10 tokens)
- Filter rate: 20 filters per minute (max 100 tokens)
- Connection rate: 1 connection per 5 minutes (max 100 tokens)

**Example from source:**
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

---

### 5. haven (Go, Khatru-based)

**Configuration File:** `.env.example`

**Default Setting:**
- No explicit limit configuration found
- Uses Khatru framework + eventstore library
- Database backend: BadgerDB or LMDB

**Behavior:**
- Inherits behavior from Khatru framework
- Uses `eventstore` library for QueryEvents implementation
- When `limit` NOT specified: **returns all matching events from database**
- **No hard cap on result size** - potentially dangerous for large datasets

**Source Code Evidence:**
```go
// init.go:134
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// Uses github.com/fiatjaf/eventstore v0.17.1
```

**Special Features:**
- Four relays in one (Private, Chat, Inbox, Outbox)
- Blossom media server
- Web of Trust filtering
- Cloud backups (S3-compatible)
- BadgerDB or LMDB storage

**Size Limits:**
- Inherits Khatru's max message size: 512,000 bytes (500 KB)

**Time Limits:**
- Import owner notes fetch timeout: 30 seconds (default)
- Import tagged notes fetch timeout: 120 seconds (default)
- Web of Trust fetch timeout: 30 seconds (default)

**Rate Limiting (per relay type):**
- Private: 50 events/interval, max 100 tokens
- Chat: 50 events/interval, max 100 tokens
- Outbox: 10 events/60s, max 100 tokens
- Inbox: 10 events/1s, max 20 tokens

**Warning:** Rate limiting applies to request rate, NOT result size. A single request without limit can still return unlimited events.

---

### 6. wot-relay (Go, Khatru-based)

**Configuration File:** `.env.example`

**Default Setting:**
- No explicit limit configuration found
- Uses Khatru framework + eventstore library
- Database backend: BadgerDB

**Behavior:**
- Inherits behavior from Khatru framework
- Uses `eventstore` library for QueryEvents implementation
- When `limit` NOT specified: **returns all matching events from database**
- **No hard cap on result size** - potentially dangerous for large datasets

**Source Code Evidence:**
```go
// main.go:145
relay.QueryEvents = append(relay.QueryEvents, db.QueryEvents)
// Uses github.com/fiatjaf/eventstore
```

**Special Features:**
- Web of Trust relay (only stores notes from followed users)
- Configurable WoT depth and minimum followers
- Optional archival sync from other relays
- Optional age-based note deletion

**Size Limits:**
- Inherits Khatru's max message size: 512,000 bytes (500 KB)

**Time Limits:**
- Maximum event age: 365 days (default, configurable via MAX_AGE_DAYS)
- Events older than MAX_AGE_DAYS are deleted

**Rate Limiting:**
```go
policies.EventIPRateLimiter(5, time.Minute*1, 30)      // 5 events/min, max 30 tokens
policies.FilterIPRateLimiter(5, time.Minute*1, 30)     // 5 filters/min, max 30 tokens
policies.ConnectionRateLimiter(10, time.Minute*2, 30)  // 10 connections/2min, max 30 tokens
```

**Configuration Options:**
```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

**Warning:** Rate limiting applies to request rate, NOT result size. A single request without limit can still return unlimited events.

---


## References

- strfry: https://github.com/hoytech/strfry
- nostream: https://github.com/cameri/nostream
- nostr-rs-relay: https://git.sr.ht/~gheartsfield/nostr-rs-relay
- khatru: https://github.com/fiatjaf/khatru
- haven: https://github.com/bitvora/haven
- wot-relay: https://github.com/bitvora/wot-relay
- eventstore: https://github.com/fiatjaf/eventstore
- nostr.watch (relay statistics): https://nostr.watch/relays/software

---

## Version Information

**Document Version:** 1.3
**Date Checked:** 2025-11-10
**Relay Versions Analyzed:**
- strfry: `542552ab0f5234f808c52c21772b34f6f07bec65` (2025-01-10)
- nostream: `6a8ccb491973b2470e3b332bef61ed7ea1143091` (2024-10-23)
- nostr-rs-relay: `d72af96d5f884e916cd3374dddd5550f8a45bfaf` (2025-02-23)
- khatru: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)
- haven: `81781b36aaafe5895bb612a452b7c0f44cde6e67` (2025-10-26)
- wot-relay: `24b51de9fdd8631f4795be85983666e76f220960` (2025-06-16)
