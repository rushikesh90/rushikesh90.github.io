---
layout: post
title: "Understanding FUSE: How SeaweedFS Becomes a POSIX Distributed Filesystem"
date: 2026-05-27
categories: [storage, distributed-systems, seaweedfs, fuse]
tags: [fuse, seaweedfs, filesystem, distributed-storage, golang]
toc: true
---

Most engineers see SeaweedFS as:

- Object Storage (S3 compatible)
- Distributed file storage
- Volume servers + filer architecture

But one of the most interesting parts is often overlooked:

**SeaweedFS can behave like a POSIX filesystem.**

You can literally do this:

```bash
weed mount \
    -dir=/mnt/sw \
    -filer=localhost:8888

cd /mnt/sw

echo hello > file.txt
mkdir docs
mv file.txt docs/
cat docs/file.txt
```

It feels like:

```text
ext4
xfs
local disk
```

But internally it is:

```text
Distributed metadata
Chunk storage
Network calls
Volume servers
Replication
```

The magic that makes this possible is **FUSE**.

---

# What is FUSE?

FUSE means:

**Filesystem in Userspace**

Normally filesystems run inside the Linux kernel.

Traditional path:

```text
Application
      │
      ▼
Linux VFS
      │
      ▼
ext4 driver
      │
      ▼
Disk
```

Example:

```bash
cat report.txt
```

Kernel directly calls ext4 code.

---

## Problem: Writing a distributed filesystem in kernel space is painful

Imagine implementing:

* Object storage
* Chunk distribution
* Replication
* Metadata synchronization
* Network protocols

inside kernel code.

Now add:

* crashes
* memory bugs
* synchronization issues

Kernel development becomes extremely expensive.

FUSE solves this by moving filesystem logic into userspace.

New flow:

```text
Application
      │
      ▼
Linux VFS
      │
      ▼
FUSE kernel module
      │
      ▼
weed mount
      │
      ▼
SeaweedFS Filer
      │
      ▼
Volume Servers
```

Filesystem logic now lives in:

```text
weed mount
```

instead of kernel code.

---

# How SeaweedFS Uses FUSE

SeaweedFS storage internally is not POSIX.

Internally a file becomes:

```text
myvideo.mp4
      │
      ▼
metadata entry
      │
      ▼
chunk references
      │
      ▼
volume server locations
```

Example:

```bash
echo hello > notes.txt
```

User sees:

```text
notes.txt
```

SeaweedFS internally sees:

```text
notes.txt
    ├── chunkA
    ├── chunkB
    └── metadata record
```

The FUSE layer translates POSIX operations.

---

## POSIX → SeaweedFS translation

Create file:

```bash
touch report.txt
```

Becomes:

```text
FUSE CREATE
        │
        ▼
weed mount
        │
        ▼
Create metadata in Filer
```

Read:

```bash
cat report.txt
```

Becomes:

```text
OPEN
READ
FETCH CHUNKS
ASSEMBLE FILE
RETURN BYTES
```

Rename:

```bash
mv a.txt b.txt
```

Becomes:

```text
Rename metadata
Update directory tree
Refresh caches
Notify clients
```

Applications never know.

To them:

```text
SeaweedFS == normal filesystem
```

---

# SeaweedFS Acting Like POSIX

POSIX means applications expect operations like:

```bash
open()
read()
write()
rename()
unlink()
mkdir()
chmod()
stat()
```

Programs assume:

```bash
cp
rsync
tar
git
vim
gcc
```

will work.

Without FUSE:

```text
Application
      │
      ▼
S3 API only
```

You would need:

```python
upload_object()
download_object()
```

With FUSE:

```bash
vim file.txt
git clone
make
tar czf
```

all become possible.

SeaweedFS suddenly behaves like:

```text
Network filesystem
```

instead of:

```text
Object store only
```

---

# Why This Makes SeaweedFS Distributed

Suppose three machines mount the same filer.

Machine A:

```bash
weed mount -dir=/mnt/sw
```

Machine B:

```bash
weed mount -dir=/mnt/sw
```

Machine C:

```bash
weed mount -dir=/mnt/sw
```

All access:

```text
/shared/project
```

Architecture:

```text
Machine A
    │
    ▼
weed mount
    │
    ├─────────────┐
    │             │
Machine B         │
    │             ▼
weed mount    SeaweedFS Filer
    │             │
Machine C         │
    │             ▼
weed mount   Volume Servers
```

Now:

Machine A:

```bash
echo test > file.txt
```

Machine B:

```bash
cat file.txt
```

Machine C:

```bash
mv file.txt done.txt
```

This is no longer local storage.

It becomes:

**Distributed POSIX access.**

---

# Why Metadata Matters

The actual bytes live in volume servers.

Example:

```text
report.pdf
```

Metadata:

```text
report.pdf
     │
     ├── chunk1 -> volume7
     ├── chunk2 -> volume2
     └── chunk3 -> volume9
```

Filer stores:

* filename
* directories
* chunk references
* attributes
* timestamps

Volume servers store:

```text
raw chunk data
```

FUSE rebuilds the file view.

Applications never see chunks.

They only see:

```bash
ls
cat
cp
mv
```

---

# Distributed Locking Becomes Necessary

Local Linux locks are insufficient.

Machine A:

```bash
flock report.db
```

Machine B:

```bash
echo overwrite > report.db
```

Kernel on machine B knows nothing.

SeaweedFS introduces distributed coordination.

Conceptually:

```text
Filer
    │
    ├── metadata
    ├── lock ownership
    └── cache state
```

Write flow:

```text
Machine A
LOCK(report.db)

Machine B
WAIT

Machine C
WAIT
```

This allows multiple mounted clients to behave as one filesystem.

---

# Why SeaweedFS Did Not Stay S3 Only

Object storage is great for:

* backups
* media
* archives
* cloud APIs

But engineers still want:

```bash
vim
git
cp
find
rsync
```

FUSE bridges these worlds.

Object storage internally:

```text
Chunks
Replication
Network storage
```

External view:

```text
POSIX filesystem
```

SeaweedFS becomes:

```text
Object Store
        +
POSIX Layer
        +
Distributed Metadata
        +
FUSE Mount
```

---

# End-to-End Example

Start cluster:

```bash
weed master

weed volume

weed filer

weed mount \
   -dir=/mnt/sw
```

Write file:

```bash
echo hello > /mnt/sw/test.txt
```

Flow:

```text
bash
   │
Linux VFS
   │
FUSE
   │
weed mount
   │
Filer metadata
   │
Volume chunks
```

Stored internally:

```text
test.txt
     │
     ├── chunkA
     └── chunkB
```

User sees:

```bash
cat test.txt

hello
```

Simple experience.

Distributed backend.

---

# Final Thoughts

FUSE is not just a mounting mechanism.

For SeaweedFS it is the layer that converts:

```text
Distributed chunk storage
```

into:

```text
POSIX filesystem semantics
```

Without FUSE:

SeaweedFS is primarily:

```text
Object storage
```

With FUSE:

SeaweedFS becomes:

```text
Distributed filesystem
```

And that is one of the reasons it can serve both:

* S3 workloads
* POSIX applications
* shared mounts
* distributed storage systems

while keeping the storage engine underneath chunk-oriented and scalable.

---

## Next Articles

1. SeaweedFS Filer Architecture
2. Chunk Storage Internals
3. Volume Server Design
4. Distributed Locking in SeaweedFS FUSE
5. Metadata Backends Explained
6. Replication and Placement Strategies