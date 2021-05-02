# Lesson 5 Consensus

## 5.1 Introduction

- What is **consensue**?
- **Theoretical limitations** on reaching consensus in distributed system.
- **FLP Theorem**
- **Practical path** toward consensus

## 5.2 What is Consensus?

#### Consensus

- Agreement among distributed process
  - on a value, action, timestamp, ...
  - on the outcome of a transaction
- Reaching a consensus makes it possible for the system to be correct
- Non-trivial for reasons related to non-determinism, lock of global clock, network delays, malicious behavior,...

#### Desired Key Properties

- All non-faulty processes eventually decide on a value **[termination/liveness]**
- All processes decide on a single value **[agreement/safety]**
- The value that'sdecided on must have been proposed by some process **[validity/safety]**
  liveness]\*\*

Can it be achieved?

## 5.3 Preliminaries: System Model

#### System Model

- **Asynchronous system**:
  - message may be reordered and delayed, but not corruputed
- At most **one faulty processor**
- **Fail-stop failure model**:
  - indistinguishable from a message delay in an asynchronous system

**Real systems are more complex**

---

Can a Consensus be Reached?
If consensus is **not possible** in this system, it will be not possible in a **system where messages** can be **corrupt**, more processor are **faulty**, or Byzantine **failures** are possible.

## 5.4 Preliminaries: Definitions

#### Some Definitions

- **Admissible run** -- run with 1 faulty processor and all messages eventutally delivered (matches system model)
- **Admissible run where some non faulty processors reach a decision** -- deciding run
- **Totally correct consensus protocol** -- if all admissible runs are also decding run
- **Uni-valent configuration** -- single decision
- **Bi-valent configuration** -- multiple/two decsions possible(non-deciding)

-

## 5.5 FLP Theorem

- M. J. Fischer, N. Lynch and M. S. Patterson, **Impossibility of distributed consensus with one faulty process**, JACM 32, 1985.
- **Dijkstra Award** for "Providing the impossibility of consensus using asynchronous communication"

> In a system with one fault, no consensus protocol can be totally correct.

## 5.6 Proof in a Nutshell

- **Start** with a system where nodes are
  - **capable of making one of two decsion, 0 or 1**
  - **1 faulty node is possbile**
  - **messages may be delayed and reordered**

1. **Lemma2**: There is an **initial configuration** for which the **final decision is not predeterminded** but **depends** on the **schedule of events** ==> There must exist an **initial bivalent configuration**
   - Also, consensus should be **made based on proposed**, not pre-determined **value**
2. A bivalent system **must go through a step** (a message being delivered) which changes it to **univalent in order to make a decsion**.
3. It is possible to **delay/reorder messages** in such a way so that the **system never goes through a bivalent -> univalent configuration change**

=> an **admissible non-deciding schedule** can exist

## 5.7 Is Consensus Really Impossible?

- **Faults** are inevitable
- **Network delays** are given
- => **cannot expect** a stronger system model
- **FLP** => Cannot have a correct distributed system!?!

---

- Some protocol
  - 2PC
  - 3PC
  - Paxos
  - Raft
- **Change** some of the **assumptions** and **system properties:**
  - **E.g.** Will the protocol (always) terminate? Under which conditions will it provide consensus?

## 5.8 Summary

## Lesson Summary

- **Explore question**: Can a distributed system **always be guaranteed** to be able to reach a consensus?
- **FLP Theorem proves** that systems with 1 faulty processor and reordered and arbitrarily delayed message, **it is impossible to guarantee that a consensus is reached**
