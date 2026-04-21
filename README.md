# PES-VCS — Version Control System in C

## Platform

Ubuntu 22.04

---

## Objective

Build a local version control system that:

* Tracks file changes
* Stores snapshots efficiently
* Supports commit history

---

## Build Instructions

```bash
make
```

---

## Usage

```bash
./pes init
./pes add <file>
./pes status
./pes commit -m "message"
./pes log
```

---

## Implementation Overview

### Object Storage (Phase 1)

* Implemented content-addressable storage using SHA-256
* Objects stored as:

  ```
  "<type> <size>\0<data>"
  ```
* Used directory sharding: `.pes/objects/XX/...`
* Ensured atomic writes using temp file + rename

---

### Tree Objects (Phase 2)

* Built tree objects from index entries
* Each tree entry contains:

  * mode
  * name
  * hash
* Serialization ensures deterministic ordering
* Stored as binary format

---

### Index / Staging Area (Phase 3)

* Text-based index file `.pes/index`
* Format:

  ```
  <mode> <hash> <mtime> <size> <path>
  ```
* Implemented:

  * index_load
  * index_save (atomic)
  * index_add
* Used metadata (mtime + size) for change detection

---

### Commits (Phase 4)

* Commit contains:

  * tree hash
  * parent hash
  * author
  * timestamp
  * message
* Stored as text object
* HEAD updated atomically
* History implemented via parent pointers

---

## Directory Structure

```
.pes/
├── objects/
├── refs/heads/
├── HEAD
└── index
```

---

## Phase 5: Branching and Checkout

### Q5.1

To implement `pes checkout <branch>`:

* Update `.pes/HEAD`:

  ```
  ref: refs/heads/<branch>
  ```
* Read commit hash from branch file
* Load commit → get tree
* Reconstruct working directory from tree:

  * write blobs to files
  * remove extra files
* Update index to match tree

**Complexity:**

* Requires full working directory reconstruction
* Must avoid overwriting user changes
* Needs careful file deletion handling

---

### Q5.2

To detect dirty working directory:

* Compare index with working directory:

  * check mtime and size
* Compare with target branch:

  * if file differs in both → conflict

**If conflict exists → abort checkout**

---

### Q5.3

Detached HEAD:

* HEAD points directly to a commit hash
* New commits are created but not referenced by any branch

**Result:**

* Commits become unreachable

**Recovery:**

* Create a branch pointing to that commit

---

## Phase 6: Garbage Collection

### Q6.1

Algorithm:

1. Start from all branch heads
2. Traverse:

   * commit → tree → blobs
3. Mark all reachable objects
4. Delete unmarked objects

**Data structure:**

* Hash set for tracking reachable hashes

**Scale:**

* ~100k commits → traverse all reachable objects

---

### Q6.2

Race condition:

* Commit creates object
* GC runs before HEAD update
* Object not marked reachable → deleted
* HEAD updated → points to missing object

**Solution (used by Git):**

* Locking mechanisms
* Safe GC timing
* Retaining recent objects (reflog)

---

## Features

* Content-addressable storage
* Deduplication via hashing
* Atomic filesystem operations
* Snapshot-based versioning
* Commit history traversal

---

## Limitations

* No branching implementation
* No merge support
* No nested directory handling (simplified tree)
* No checkout command implemented

---

## Author

Set using:

```bash
export PES_AUTHOR="Your Name <SRN>"
```

---

## Conclusion

This project demonstrates how version control systems like Git work internally using:

* hashing
* filesystem structures
* immutable objects
* linked commit history

---
