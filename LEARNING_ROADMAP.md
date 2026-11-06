# Learning Roadmap: Databases, Distributed Systems, Staff-Engineer Mindset

Goal: understand how SurrealDB works, become strong in distributed systems
design, and contribute to SurrealDB.

## Key fact before starting

This repo has **no consensus code**. Its storage backends are only `kv-mem`
and `kv-surrealkv` (see `surrealdb/kvs-any/Cargo.toml`). In distributed
deployments SurrealDB relies on the store underneath (historically TiKV,
which uses Raft) for replication and consensus.

So learning is split across two places:

- **This repo:** database engines and coordination *above* consensus:
  transactions, MVCC, storage formats, query planning, node liveness,
  change feeds, upgrade compatibility.
- **Outside this repo:** consensus and replication (Raft, quorums,
  linearizability) via etcd/raft, TiKV, and building it yourself.

---

## Phase 0: Foundations (weeks 0–4, alongside Phase 1)

- [ ] *Designing Data-Intensive Applications* (Kleppmann): chapters 3, 5, 7, 8, 9
- [ ] Rust: async/await, Tokio, traits, lifetimes
- [ ] Build and run the dev server with trace logging:
  ```bash
  cargo run --locked --no-default-features --features storage-mem,http,scripting -- start --log trace --user root --pass root memory
  ```
- [ ] Run queries and read the trace output

## Phase 1: Follow one query end to end (weeks 1–4)

Trace `CREATE person:1 SET name='x'; SELECT * FROM person;` through every layer.

| Layer | Where |
|---|---|
| Parser | `surrealdb/parser-next/` (read its `CLAUDE.md`) |
| Logical/physical plan | `surrealdb/core/src/exec/planner/`, `exec/logical/` |
| Execution (bytecode VM) | `surrealdb/core/src/exec/vm/` |
| Key encoding | `surrealdb/core/src/key/`, `surrealdb/kvs/src/key/` |
| Transaction API | `surrealdb/kvs/src/api.rs`, `isolation.rs`, `savepoint.rs`, `snapshot.rs` |
| Storage engine | `surrealdb/kvs-surrealkv/src/` + upstream `surrealkv` crate (LSM tree) |

- [ ] Read all ADRs in `doc/adr/`. For each one, write down the rejected
      alternatives and what would make the decision wrong.
- [ ] Trace the CREATE query end to end
- [ ] Trace the SELECT query end to end

## Phase 2: Transactions, MVCC, storage (weeks 4–8)

- [ ] `surrealdb/kvs/README.md` + `src/`: what a backend must guarantee vs.
      what is optional (`snapshot`, `estimate`, `split`)
- [ ] `kvs/src/timestamp.rs`, `core/src/kvs/clock.rs`: how versions and
      timestamps are assigned; why clocks are hard
- [ ] **Build:** a small MVCC KV store in Rust with snapshot isolation and
      write-write conflict detection. Compare with surrealkv.
- [ ] Papers: Bigtable, the LSM-tree paper, "A Critique of ANSI SQL Isolation Levels"

## Phase 3: Distributed coordination in this repo (weeks 8–12)

- [ ] **Node liveness and cleanup:** `core/src/kvs/ds.rs`. Search
      `node_heartbeat`, `node_cleanup_*`, `nodes_without_row`.
      Question: how does the cluster detect a dead node and recover its
      state (e.g. live queries) without a coordinator?
- [ ] **Change feeds:** `core/src/cf/` (`writer`, `reader`, `gc`). Ordered
      change capture, a log you can replicate.
- [ ] **Background tasks:** `core/src/kvs/tasks/`. GC and paged work that
      resumes after a crash (see the `*_is_paged_across_passes` tests).
- [ ] **Live queries:** `core/src/exec/live.rs`, `core/src/lq/`
- [ ] **Upgrade safety:** `language-tests/tests/upgrade/README.md`. Data
      written by an old binary must stay readable by every later one.

## Phase 4: Consensus, outside this repo (weeks 8–16)

- [ ] Raft paper ("In Search of an Understandable Consensus Algorithm")
- [ ] Raft thesis, chapters 3–6
- [ ] **MIT 6.5840 Labs 2–4:** Raft → KV store on Raft → sharding (do not skip)
- [ ] Read `tikv/raft-rs` (Rust) or `etcd-io/raft` (Go)
- [ ] Jepsen analyses (jepsen.io): etcd, TiKV, CockroachDB
- [ ] Papers: Spanner, Percolator (TiKV's transaction model), Calvin

## Phase 5: Contributing (start in week 2, keep going)

- [ ] Read `CONTRIBUTING.md`, `REVIEW.md`, `SECURITY_GUIDE.md`, and the
      "Comment discipline" section of `CLAUDE.md`
- [ ] **First contributions = reproductions.** Take an open bug, write
      `language-tests/tests/reproductions/ISSUE_summary.surql`
      (format in `language-tests/CLAUDE.md`), decide if it's a real bug
- [ ] Fix a small bug inside one module (parser edge case, a function)
- [ ] Move to storage/tasks work: paging, GC, cleanup edge cases
- [ ] Before every PR: `cargo make fmt`, `cargo make ci-clippy`, `--locked` on cargo commands
- [ ] Find work: `gh issue list -R surrealdb/surrealdb --label bug`.
      Comment before starting anything large.

## Staff-engineer habits (ongoing)

- **Write design docs.** For each subsystem: problem, constraints, options,
  choice, what breaks it. Compare yours with the real ADR.
- **Think failure first.** For every code path: crash here? clock jumps?
  two nodes at once?
- **Read maintainer PR reviews** on `surrealdb/surrealdb`.
- **Measure.** Use `profiling/` and `surrealdb/benches/`. No speed claims
  without numbers.
- **Teach it.** Write up each phase. If you can't explain how node cleanup
  survives a crash mid-pass, you don't understand it yet.

---

## Week 1 checklist

- [ ] Run the server with `--log trace`
- [ ] Read ADRs 0001, 0004, 0009
- [ ] Trace one `SELECT` from parser to `kvs`
- [ ] Set up the MIT 6.5840 Lab 2 skeleton

## Progress log

<!-- Add a dated line each session: what you read, what you learned, open questions. -->

| Date | What I did | Open questions |
|---|---|---|
| 2026-11-06 | Created roadmap | |
