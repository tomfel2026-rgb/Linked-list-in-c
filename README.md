# C-Linked-List: Internal Logic & Memory Mapping

This document serves as a deep dive into the implementation details of this library. If you are getting `Segfaults`, read this before you start crying in the issues tab.

---

## 🧠 How the Code Works (The "Magic" Explained)

### 1. Node Anatomy & Memory Allocation
Each element in this list is not just "data"; it is a dynamically allocated block of memory on the **Heap**. When you call `push()`, the code performs the following dance:

1.  **Request Space**: `malloc(sizeof(Node))` asks the OS for a small chunk of memory.
2.  **Initialization**: The `data` field is populated, and—crucially—the `next` pointer is set to `NULL` to prevent it from pointing to random garbage in RAM.
3.  **Linking**: The new node’s `next` pointer is assigned the address of the current `head`, and then the `head` is updated to the new node's address.

### 2. The Logic of Traversal
Since a Singly Linked List is **not** contiguous in memory (unlike an array), we cannot use index math (`list[5]`). Instead, the code uses a "Current" pointer:
- It starts at `head`.
- It enters a `while(current != NULL)` loop.
- It "jumps" from one node to the next using `current = current->next`.

### 3. Deletion & Pointer Re-linking
This is where most people (probably you) break the chain. To delete a node:
1.  We find the node **before** the one we want to kill.
2.  We take that "Previous" node's `next` pointer and point it to the "Target" node's `next`.
3.  **CRITICAL**: Only after the chain is re-linked do we call `free()` on the target. If you `free()` first, you lose the rest of your list. Forever.

---

## 🖼️ Memory Visualization

A 3-node list (10 -> 20 -> 30) looks like this in your RAM:



```text
Stack          Heap Memory
+-------+      +---------------+       +---------------+       +---------------+
| head  | ---> | Data: 10      |       | Data: 20      |       | Data: 30      |
+-------+      | Next: 0x0042  | --->  | Next: 0x0084  | --->  | Next: NULL    |
               +---------------+       +---------------+       +---------------+
               Addr: 0x0021            Addr: 0x0042            Addr: 0x0084
