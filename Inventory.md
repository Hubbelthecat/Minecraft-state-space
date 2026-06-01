# Inventory State Space Model (Java Edition 1.21.11)

This document defines the formal mathematical model for the Minecraft inventory state space.

It describes how individual item slots, multi-slot systems, and recursive container structures contribute to the total state space.

---

# 1. Inventory State Definition

An inventory state is defined as any configuration that can be distinguished by:

* Game engine logic
* Memory representation
* NBT data
* Redstone behavior
* Comparators
* Packets
* Commands
* Any other mechanically detectable system

A distinct state exists only if it is:

> Persistently stored and distinguishable by the game as a valid inventory configuration.

---

# 2. Slot Model Definition

## 2.1 State Variable

We define:

Ω_slot = number of possible states of a single inventory slot

This is the fundamental unit of the model.

---

# 3. Single Slot Models

## 3.1 One Slot, One Item Type

Stack size = 64

States:

* Empty
* 1 item
* ...
* 64 items

Ω = 65

H = log₂(65)

H ≈ 6.022 bits

bits_needed = 7

---

## 3.2 One Slot, 1,238 Item Types

Ω = 1238 × 64

Ω = 79,232

H ≈ 16.28 bits

log₂(1238) ≈ 10.28 bits
log₂(64) = 6 bits

bits_needed = 17

---

## 3.3 Full Vanilla Item System (No NBT)

Item distribution:

* 1,238 items stack to 64
* 26 items stack to 16
* 225 items stack to 1
* Empty slot

Ω_slot defined as:

Ω_slot =
1238 × 64 +
26 × 16 +
225 × 1 +
1

Ω_slot = 79,874

H = log₂(79,874)

H ≈ 16.29 bits

bits_needed = 17

---

# 4. Multi-Slot System

## 4.1 General Rule

For n independent slots:

Ω_total = Ω_slotⁿ

H_total = n × log₂(Ω_slot)

---

## 4.2 Two Slots

Ω_total = 79,874²
Ω_total = 6,379,859,876

H ≈ 32.58 bits
bits_needed = 33

---

## 4.3 Five Slots

Ω_total = Ω_slot⁵

H ≈ 81.45 bits
bits_needed = 82

---

## 4.4 Standard Inventory (45 Slots)

Slot composition:

* 36 inventory
* 1 off-hand
* 4 crafting
* 4 armor slots (unrestricted via commands)

Ω_total = Ω_slot⁴⁵

Ω_total = 79,874⁴⁵

Ω_total ≈ 10²²¹

H ≈ 733.05 bits
bits_needed = 734 (~92 bytes)

---

# 5. Recursive Container Expansion (Shulkers)

## 5.1 Definition

k = number of shulker boxes

Each shulker:

* Consumes 1 slot
* Adds 27 internal slots

Net gain:

+26 slots per shulker

Effective slot function:

S(k) = 45 + 26k

---

## 5.2 Recursive State Space

Ω(k) = Ω_slot^(45 + 26k)

Ω(k) = 79,874^(45 + 26k)

Total system:

Ω_total = Σ 79,874^(45 + 26k), k = 0 → 45

---

## 5.3 Maximum Topology

S_max = 1215 slots

Ω_total ≈ 79,874^1215

Ω_total ≈ 10^5957

H ≈ 19,792 bits
≈ 2.47 KB

---

# 6. Bundle Model (Approximation)

## 6.1 Configuration Space

Ω_bundle ≈ C(1301,64)

Approximation:

Ω_bundle ≈ 10^106

H_bundle ≈ 352 bits
≈ 44 bytes

---

## 6.2 Constraints

* Bundles cannot contain bundles
* Bundles cannot contain shulkers
* Shulkers cannot contain shulkers
* Shulkers may contain bundles

---

## 6.3 Binary Approximation

Ω_bundle ≈ 2^352

---

# 7. Full Recursive Inventory Model

## 7.1 State Space

Ω(k) ≈ (2^352)^(45 + 26k)

Simplified:

Ω(k) ≈ 2^(352(45 + 26k))

Total:

Ω_total = Σ 2^(352(45 + 26k)), k = 0 → 45

---

## 7.2 Maximum Topology

1215 effective slots

Ω_total ≈ 2^427,680

Ω_total ≈ 10^128,732

H ≈ 427,680 bits
≈ 53.5 KB

---

# 8. Cursor Slot Correction

## 8.1 Observation

The inventory cursor is a fully valid independent slot.

It can contain:

* Items
* Bundles
* Shulker boxes
* Nested shulker structures

Therefore it contributes fully to state space.

---

## 8.2 Corrected Slot Count

Total slots:

* 36 inventory
* 1 off-hand
* 4 crafting
* 4 armor
* 1 cursor

Total:

46 slots

---

## 8.3 Corrected Expansion

S(k) = 46 + 26k

k = 46

S_max = 1242

---

## 8.4 Corrected Final Model

Ω_total ≈ 79,874^1242

Ω_total ≈ 10^6088

H ≈ 20,232 bits
≈ 2.53 KB

---

## 8.5 Final Recursive Model

Ω_total ≈ 2^437,184

Ω_total ≈ 10^131,593

H ≈ 437,184 bits
≈ 54.6 KB

---

# 9. Key Insight

Adding a single recursive slot increases not only the state space linearly, but also expands the entire combinatorial structure of all nested inventory configurations.

This creates exponential branching across all recursive container systems.

