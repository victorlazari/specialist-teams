# Valkey / Redis Specialist Guide

## 1. Overview and Core Concepts

Valkey is an open source (BSD licensed), in-memory data structure store used as a database, cache, message broker, and streaming engine [1]. It is a high-performance alternative to Redis, maintaining compatibility while being fully open source under the Linux Foundation. Valkey provides a wide array of data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams [1].

Valkey has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence [1]. It provides high availability via Valkey Sentinel and automatic partitioning with Valkey Cluster [1]. 

## 2. Architecture and Data Structures

To achieve top performance, Valkey works with an in-memory dataset [1]. Depending on the use case, Valkey can persist data either by periodically dumping the dataset to disk or by appending each command to a disk-based log [1]. Persistence can also be disabled if a feature-rich, networked, in-memory cache is all that is required [1].

Valkey supports asynchronous replication, with fast non-blocking synchronization and auto-reconnection with partial resynchronization on net split [1].

Valkey is written in ANSI C 11 with Atomics and a few GCC/Clang built-ins. It works on most POSIX systems like Linux, *BSD and MacOS, without external dependencies. Linux is the recommended operating system for deployment [1].

## 3. Advanced Features

For deeper dives into advanced topics, please refer to the child document: `valkey-redis-advanced.md`. The advanced document covers:
- High Availability with Valkey Sentinel
- Horizontal Scaling with Valkey Cluster
- Advanced Configuration and Performance Tuning
- Troubleshooting and Diagnosing Latency Issues

## References
[1] [Valkey Documentation: Introduction](https://valkey.io/topics/introduction/)
