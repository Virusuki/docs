---
Title: Redis Community Edition 7.4 release notes
alwaysopen: false
categories:
- docs
- operate
- stack
description: Redis Community Edition 7.4 release notes.
linkTitle: v7.4.0 (July 2024)
min-version-db: blah
min-version-rs: blah
weight: 100
---

## Redis Community Edition 7.4.5 (July 2025)

Update urgency: `SECURITY`: There are security fixes in the release.

### Security fixes

* (CVE-2025-27151) redis-check-aof may lead to stack overflow and potential RCE

### Bug fixes

- [#13966](https://github.com/redis/redis/pull/13966), [#13932](https://github.com/redis/redis/pull/13932) `CLUSTER SLOTS` - TLS port update not reflected in CLUSTER SLOTS
- [#13958](https://github.com/redis/redis/pull/13958) `XTRIM`, `XADD` - incorrect lag due to trimming stream

## Redis Community Edition 7.4.4 (May 2025):

Update urgency: `SECURITY`: There are security fixes in the release.

### Security fixes

* (CVE-2025-27151) redis-check-aof may lead to stack overflow and potential RCE.

### Bug fixes

- [#13966](https://github.com/redis/redis/pull/13966), [#13932](https://github.com/redis/redis/pull/13932) `CLUSTER SLOTS` - TLS port update not reflected in CLUSTER SLOTS.
- [#13958](https://github.com/redis/redis/pull/13958) `XTRIM`, `XADD` - incorrect lag due to trimming stream.

## Redis Community Edition 7.4.3 (April 2025)

Update urgency: `SECURITY`: There are security fixes in the release.

### Security fixes

* (CVE-2025-21605) An unauthenticated client can cause an unlimited growth of output buffers.

### Bug fixes

* [#13661](https://github.com/redis/redis/pull/13661) `FUNCTION FLUSH` - memory leak when using jemalloc.
* [#13793](https://github.com/redis/redis/pull/13793) `WAITAOF` returns prematurely.
* [#13853](https://github.com/redis/redis/pull/13853) `SLAVEOF` - crash when clients are blocked on lazy free.
* [#13863](https://github.com/redis/redis/pull/13863) `RANDOMKEY` - infinite loop during client pause.
* [#13877](https://github.com/redis/redis/pull/13877) ShardID inconsistency when both primary and replica support it.

## Redis Community Edition 7.4.2 (Jan 2025)

Upgrade urgency SECURITY: See security fixes below.

### Security fixes

- (CVE-2024-46981) Lua script commands may lead to remote code execution
- (CVE-2024-51741) Denial-of-service due to malformed ACL selectors

### Bug fixes

- [#13627](https://github.com/redis/redis/pull/13627) Crash on module memory defragmentation
- [#13338](https://github.com/redis/redis/pull/13338) Streams: `XINFO` lag field is wrong when tombstone is after the `last_id` of the consume group
- [#13473](https://github.com/redis/redis/pull/13473) Streams: `XTRIM` does not update the maximal tombstone, leading to an incorrect lag
- [#13470](https://github.com/redis/redis/pull/13470) `INFO` after `HDEL` show wrong number of hash keys with expiration
- [#13476](https://github.com/redis/redis/pull/13476) Fix a race condition in the `cache_memory` of `functionsLibCtx`
- [#13626](https://github.com/redis/redis/pull/13626) Memory leak on failed RDB loading
- [#13539](https://github.com/redis/redis/pull/13539) Hash: fix key ref for a hash that no longer has fields with expiration on `RENAME`/`MOVE`/`SWAPDB`/`RESTORE`
- [#13443](https://github.com/redis/redis/pull/13443) Cluster: crash when loading cluster config
- [#13422](https://github.com/redis/redis/pull/13422) Cluster: `CLUSTER SHARDS` returns empty array
- [#13465](https://github.com/redis/redis/pull/13465) Cluster: incompatibility with older node versions
- [#13608](https://github.com/redis/redis/pull/13608) Cluster: `SORT ... GET #`: incorrect error message

## Redis Community Edition 7.4.1 (October 2024)

Upgrade urgency SECURITY: See security fixes below.

### Security fixes
* (CVE-2024-31449) Lua library commands may lead to stack overflow and potential RCE.
* (CVE-2024-31227) Potential Denial-of-service due to malformed ACL selectors.
* (CVE-2024-31228) Potential Denial-of-service due to unbounded pattern matching.

For more information, see the [Redis blog post](https://redis.io/blog/security-advisory-cve-2024-31449-cve-2024-31227-cve-2024-31228/) about these vulnerabilities.

## Redis Community Edition 7.4 (July 2024)

This is the General Availability release of Redis Community Edition 7.4.

**Changes to new 7.4 features (compared to 7.4 RC2)**
* [#13391](https://github.com/redis/redis/pull/13391),[#13438](https://github.com/redis/redis/pull/13438) Hash - expiration of individual fields: RDB file format changes
* [#13372](https://github.com/redis/redis/pull/13372) Hash - expiration of individual fields: rename and fix counting of `expired_subkeys` metric
* [#13372](https://github.com/redis/redis/pull/13372) Hash - expiration of individual fields: rename `INFO` keyspace field to `subexpiry`

**Configuration parameters**
* [#13400](https://github.com/redis/redis/pull/13400) Add hide-user-data-from-log - allows hiding user data from the log file

**Bug fixes**
* [#13407](https://github.com/redis/redis/pull/13407) Trigger Lua GC after `SCRIPT LOAD`
* [#13380](https://github.com/redis/redis/pull/13380) Fix possible crash due to OOM panic on invalid command
* [#13383](https://github.com/redis/redis/pull/13383) `FUNCTION FLUSH` - improve Lua GC behavior and fix thread race in ASYNC mode
* [#13408](https://github.com/redis/redis/pull/13408) `HEXPIRE`-like commands should emit `HDEL` keyspace notification if expire time is in the past

## Redis Community Edition 7.4-rc2 (June 2024)
Upgrade urgency LOW: This is the second Release Candidate for Redis Community Edition 7.4.

**Performance and resource utilization improvements**
* [#13296](https://github.com/redis/redis/pull/13296) Optimize CPU cache efficiency

**Changes to new 7.4 new features (compared to 7.4 RC1)**
* [#13343](https://github.com/redis/redis/pull/13343) Hash - expiration of individual fields: when key does not exist - reply with an array (nonexisting code for each field)
* [#13329](https://github.com/redis/redis/pull/13329) Hash - expiration of individual fields: new keyspace event: `hexpired`

**Modules API - Potentially breaking changes to new 7.4 features (compared to 7.4 RC1)**
* [#13326](https://github.com/redis/redis/pull/13326) Hash - expiration of individual fields: avoid lazy expire when called from a Modules API function

## Redis Community Edition 7.4-rc1 (June 2024)

Upgrade urgency LOW: This is the first Release Candidate for Redis Community Edition 7.4.

Here is a comprehensive list of changes in this release compared to 7.2.5.

**New Features**
* [#13303](https://github.com/redis/redis/pull/13303) Hash - expiration of individual fields. 9 commands were introduced:
  - `HEXPIRE` and `HPEXPIRE` set the remaining time to live for specific fields
  - `HEXPIREAT` and `HPEXPIREAT` set the expiration time to a UNIX timestamp for specific fields
  - `HPERSIST` removes the expiration for specific fields
  - `HEXPIRETIME` and `HPEXPIRETIME` get the expiration time for specific fields
  - `HTTL` and `HPTTL` get the remaining time to live for specific fields
* [#13117](https://github.com/redis/redis/pull/13117) `XREAD`: new id value `+` to start reading from the last message
* [#12765](https://github.com/redis/redis/pull/12765) `HSCAN`: new `NOVALUES` flag to report only field names
* [#12728](https://github.com/redis/redis/pull/12728) `SORT`, `SORT_RO`: allow `BY` and `GET` options in cluster mode when the pattern maps to the same slot as the key
* [#12299](https://github.com/redis/redis/pull/12299) `CLIENT KILL`: new optional filter: `MAXAGE maxage` - kill connections older than `maxage` seconds
* [#12971](https://github.com/redis/redis/pull/12971) Lua: expose `os.clock()` API for getting the elapsed time of Lua code execution
* [#13276](https://github.com/redis/redis/pull/13276) Allow `SPUBLISH` command within `MULTI ... EXEC` transactions on replica

**Bug fixes**
* [#12898](https://github.com/redis/redis/pull/12898) `XREADGROUP`: fix entries-read inconsistency between master and replicas
* [#13042](https://github.com/redis/redis/pull/13042) `SORT ... STORE`: fix created lists to respect list compression and packing configs
* [#12817](https://github.com/redis/redis/pull/12817), [#12905](https://github.com/redis/redis/pull/12905) Fix race condition issues between the main thread and module threads
* [#12577](https://github.com/redis/redis/pull/12577) Unsubscribe all clients from replica for shard channel if the master ownership changes
* [#12622](https://github.com/redis/redis/pull/12622) `WAITAOF` could timeout or hang if used after a module command that propagated effects only to replicas and not to AOF
* [#11734](https://github.com/redis/redis/pull/11734) `BITCOUNT` and `BITPOS` with nonexistent key and illegal arguments return an error, not 0
* [#12394](https://github.com/redis/redis/pull/12394) `BITCOUNT`: check for wrong argument before checking if key exists
* [#12961](https://github.com/redis/redis/pull/12961) Allow execution of read-only transactions when out of memory
* [#13274](https://github.com/redis/redis/pull/13274) Fix crash when a client performs ACL change that disconnects itself
* [#13311](https://github.com/redis/redis/pull/13311) Cluster: Fix crash due to unblocking client during slot migration

**Security improvements**
* [#13108](https://github.com/redis/redis/pull/13108) Lua: LRU eviction for scripts generated with `EVAL` *** BEHAVIOR CHANGE ***
* [#12961](https://github.com/redis/redis/pull/12961) Restrict the total request size of `MULTI ... EXEC` transactions
* [#12860](https://github.com/redis/redis/pull/12860) Redact ACL username information and mark `*-key-file-pass configs` as sensitive


**Performance and resource utilization improvements**
* [#12838](https://github.com/redis/redis/pull/12838) Improve performance when many clients call `PUNSUBSCRIBE` / `SUNSUBSCRIBE` simultaneously
* [#12627](https://github.com/redis/redis/pull/12627) Reduce lag when waking `WAITAOF` clients and there is not much traffic
* [#12754](https://github.com/redis/redis/pull/12754) Optimize `KEYS` when pattern includes hashtag and implies a single slot
* [#11695](https://github.com/redis/redis/pull/11695) Reduce memory and improve performance by replacing cluster metadata with slot specific dictionaries
* [#13087](https://github.com/redis/redis/pull/13087) `SCRIPT FLUSH ASYNC` now does not block the main thread
* [#12996](https://github.com/redis/redis/pull/12996) Active memory defragmentation efficiency improvements
* [#12899](https://github.com/redis/redis/pull/12899) Improve performance of read/update operation during rehashing
* [#12536](https://github.com/redis/redis/pull/12536) `SCAN ... MATCH`: Improve performance when the pattern implies cluster slot
* [#12450](https://github.com/redis/redis/pull/12450) `ZRANGE ... LIMIT`: improved performance


**Other general improvements**
* [#13133](https://github.com/redis/redis/pull/13133) Lua: allocate VM code with jemalloc instead of libc and count it as used memory *** BEHAVIOR CHANGE ***
* [#12171](https://github.com/redis/redis/pull/12171) `ACL LOAD`: do not disconnect all clients *** BEHAVIOR CHANGE ***
* [#13020](https://github.com/redis/redis/pull/13020) Allow adjusting defrag configurations while active defragmentation is running
* [#12949](https://github.com/redis/redis/pull/12949) Increase the accuracy of avg_ttl (the average keyspace keys TTL)
* [#12977](https://github.com/redis/redis/pull/12977) Allow running `WAITAOF` in scripts
* [#12782](https://github.com/redis/redis/pull/12782) Implement TCP Keep-Alives across most Unix-like systems
* [#12707](https://github.com/redis/redis/pull/12707) Improved error codes when rejecting scripts in cluster mode
* [#12596](https://github.com/redis/redis/pull/12596) Support `XREAD ... BLOCK` in scripts; rejected only if it ends up blocking

**New metrics**
* [#12849](https://github.com/redis/redis/pull/12849) `INFO`: `pubsub_clients` - number of clients in Pub/Sub mode
* [#12966](https://github.com/redis/redis/pull/12966) `INFO`: `watching_clients` - number of clients that are watching keys
* [#12966](https://github.com/redis/redis/pull/12966) `INFO`: `total_watched_keys` - number of watched keys
* [#12476](https://github.com/redis/redis/pull/12476) `INFO`: `client_query_buffer_limit_disconnections` - count client input buffer OOM events
* [#12476](https://github.com/redis/redis/pull/12476) `INFO`: `client_output_buffer_limit_disconnections` - count client output buffer OOM events
* [#12996](https://github.com/redis/redis/pull/12996) `INFO`: `allocator_muzzy` - memory returned to the OS but still shows as RSS until the OS reclaims it
* [#13108](https://github.com/redis/redis/pull/13108) `INFO`: `evicted_scripts` - number of evicted eval scripts. Users can check it to see if they are abusing EVAL
* [#12996](https://github.com/redis/redis/pull/12996) `MEMORY STATS`: `allocator.muzzy` - memory returned to the OS but still shows as RSS until the OS reclaims it
* [#12913](https://github.com/redis/redis/pull/12913) `INFO MEMORY` `mem_overhead_db_hashtable_rehashing` - memory resharding overhead (only the memory that will be released soon)
* [#12913](https://github.com/redis/redis/pull/12913) `MEMORY STATS`: `overhead.db.hashtable.lut` - total overhead of dictionary buckets in databases
* [#12913](https://github.com/redis/redis/pull/12913) `MEMORY STATS`: `overhead.db.hashtable.rehashing` - temporary memory overhead of database dictionaries currently being rehashed 
* [#12913](https://github.com/redis/redis/pull/12913) `MEMORY STATS`: `db.dict.rehashing.count` - number of top level dictionaries currently being rehashed
* [#12966](https://github.com/redis/redis/pull/12966) `CLIENT LIST`: `watch` - number of keys each client is currently watching

**Modules API**
* [#12985](https://github.com/redis/redis/pull/12985) New API calls: `RM_TryCalloc` and `RM_TryRealloc` - allow modules to handle memory allocation failures gracefully
* [#13069](https://github.com/redis/redis/pull/13069) New API call: `RM_ClusterKeySlot` - which slot a key will hash to
* [#13069](https://github.com/redis/redis/pull/13069) New API call: `RM_ClusterCanonicalKeyNameInSlot` - get a consistent key that will map to a slot 
* [#12486](https://github.com/redis/redis/pull/12486) New API call: `RM_AddACLCategory` - allow modules to declare new ACL categories


**Configuration parameters**
* [#12178](https://github.com/redis/redis/pull/12178) New configuration parameters: `max-new-connections-per-cycle` and `max-new-tls-connections-per-cycle` to limit the number of new client connections per event-loop cycle
* [#7351](https://github.com/redis/redis/pull/7351) Rename some CPU configuration parameters for style alignment. Added alias to the old names to avoid breaking change

**CLI tools**
* [#10609](https://github.com/redis/redis/pull/10609) redis-cli: new `-t <timeout>` argument: specify server connection timeout in seconds
* [#11315](https://github.com/redis/redis/pull/11315) redis-cli: new `-4` and `-6` flags to prefer IPV4 or IPV6 on DNS lookup
* [#12862](https://github.com/redis/redis/pull/12862) redis-cli: allows pressing up arrow to return any command (including sensitive commands which are still not persisted)
* [#12543](https://github.com/redis/redis/pull/12543) redis-cli: add reverse history search (like Ctrl+R in terminals)
* [#12826](https://github.com/redis/redis/pull/12826) redis-cli: add `--keystats` and `--keystats-samples` to combines `--memkeys` and `--bigkeys` with additional distribution data
* [#12735](https://github.com/redis/redis/pull/12735) redis-cli: fix: `--bigkeys` and `--memkeys` now work on cluster replicas
* [#9411](https://github.com/redis/redis/pull/9411) redis-benchmark: add support for binary strings
* [#12986](https://github.com/redis/redis/pull/12986) redis-benchmark: fix: pick random slot for a node to distribute operation across slots
