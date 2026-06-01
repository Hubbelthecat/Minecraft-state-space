# Inventory State Definition

If anything about an inventory can be distinguished by the game engine, memory, NBT, redstone behavior, comparators, packets, commands, or any other detectable mechanic, it counts as a separate state.

A distinct inventory state exists only if the game stores it as a persistent, serialized, distinguishable data state attached to the item or its container slot.

---

# Single Slot Models

## One Slot, One Item Type

Stack size: 64

Possible states:

* Empty
* 1 item
* 2 items
* ...
* 64 items

State space:

Ω = 65

Entropy:

H = log₂(65)

H ≈ 6.022 bits

Storage:

bits_needed = 7

---

## One Slot, 1,238 Item Types

Stack size: 64

Each state:

(Item Type, Item Count)

State space:

Ω = 1238 × 64

Ω = 79,232

Entropy:

H ≈ 16.28 bits

Breakdown:

log₂(1238) ≈ 10.28 bits

log₂(64) = 6 bits

Storage:

bits_needed = 17

---

## Full Vanilla Item System (No NBT)

Item counts:

* 1,238 items stack to 64
* 26 items stack to 16
* 225 items stack to 1
* Empty slot

State space:

Ω_slot =
1238 × 64 +
26 × 16 +
225 × 1 +
1

Ω_slot = 79,874

Entropy:

H = log₂(79,874)

H ≈ 16.29 bits

Storage:

bits_needed = 17

Definition:

Ω_slot = 79,874

This is the state space of a single inventory slot.

---

# Multi-Slot Inventories

## Two Independent Slots

Ω_total = Ω_slot²

Ω_total = 79,874²

Ω_total = 6,379,859,876

Entropy:

H = 2 × log₂(79,874)

H ≈ 32.58 bits

Storage:

bits_needed = 33

---

## Five Independent Slots

Ω_total = Ω_slot⁵

Entropy:

H = 5 × log₂(79,874)

H ≈ 81.45 bits

Storage:

bits_needed = 82

---

## Standard Inventory (45 Slots)

Slots:

* 36 inventory
* 1 off-hand
* 4 crafting
* 4 armor slots (unrestricted via commands)

State space:

Ω_total = Ω_slot⁴⁵

Ω_total = 79,874⁴⁵

Scientific notation:

Ω_total ≈ 10²²¹

Entropy:

H ≈ 733.05 bits

Storage:

bits_needed = 734

≈ 92 bytes

---

