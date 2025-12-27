# Runtime-Decrypted Execution Pipeline (C, macOS ARM64)

This project actually started very small.

I was just playing with **message / data-level obfuscation**, like using `memfrob()` which is basically XOR-ing bytes.  
For example, take the string `"shubham"` and XOR every byte with `0x2A` (42).

At first, it looks confusing.  
But very soon it becomes obvious that:

- it is easy to reverse
- memory inspection or tracing shows exactly what’s happening
- most importantly, you are hiding **data**, not **logic**

That’s when I realised the real problem isn’t the string itself —  
the real problem is **where and when the code exists**.

So instead of hiding data, I tried hiding **execution logic**.

---

## What I Built

A **runtime-decrypted execution pipeline in C** (macOS, ARM64).

The idea is simple but powerful:

- the payload is stored **encrypted** inside the binary
- **RW (read–write) memory** is allocated at runtime using `mmap` (requested from the kernel)
- the code is decrypted **only inside that memory**
- memory permissions are switched from **RW → RX** (W^X enforcement)
- execution happens via a **function pointer**
- after execution, memory is **wiped and unmapped immediately**
- a **runtime XOR key changes on every run** (polymorphism)

---

## What This Achieves

Because of this design:

- static analysis shows **no embedded payload**
- memory dumps **after execution** show nothing useful
- only loader logic (`mmap`, `mprotect`, `memcpy`, etc.) is visible
- instruction bytes are **not stable across runs**

Plaintext instructions exist **only for a very small execution window**.

---

## Common Confusion

> “But 1337 is already visible when you run the program.”

Yes — and that’s expected.

If a program is allowed to print something, that output is **not the secret**.

The goal here is **not** to hide output.  
The goal is to hide **how that output is produced**.

In real systems:
- secrets control behavior
- they are not meant to be printed

This project focuses on **protecting logic**, not hiding user-visible output.

---

## What I Learned

- the difference between **data-level** and **code-level** obfuscation
- why **W^X** exists and how the OS enforces it
- why CPUs don’t care *where* code comes from
- why **timing matters** (when plaintext exists in memory)
- how **polymorphism breaks determinism**
- why static analysis fails, but also why dynamic analysis still matters

Most importantly:

> **Security is not about making things impossible —  
> it’s about making analysis expensive.**

---

## Proofs

The following proofs are included using standard tools:

- `strings` → no plaintext payload
- `otool` → no static comparison or embedded instructions
- `lldb` → runtime-only execution and memory wiping


