# Chunks

## Introduction

A chunk divides a large region of memory into chunks of various sizes. They are stored in contiguous regions of memory.

There are four types of chunks:

1. Top chunk - the chunk that is at the top border of an arena
2. In-use chunks
3. Free chunks
4. Last remainder chunk

No two free chunks can be adjacent together. When both chunks are free, they get coalesced into one single free chunk.
