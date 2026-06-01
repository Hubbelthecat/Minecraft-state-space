# State Space vs Entropy

## State Space

The State Space of a system is the set of all possible states the system can occupy.

In this project, the state space size is written as:

Ω (Omega)

where:

Ω = total number of possible microstates

A microstate is one exact configuration of the system.

Examples:
- A specific inventory arrangement.
- A particular set of NBT data.
- A specific Minecraft world configuration.

The state space answers the question:

"How many different exact configurations are possible?"

State space is the primary quantity being calculated throughout this project.

---

## Entropy

Entropy is a logarithmic measure of the state space.

For information entropy:

H = log₂(Ω)

where:

H = entropy in bits
Ω = number of possible states


Entropy answers the question:

"How much information is required to uniquely identify a state?"

Because entropy uses a logarithm, it compresses extremely large state spaces into more manageable numbers.

Example:

Ω = 1,024

H = log₂(1,024) = 10 bits

This means:
- There are 1,024 possible states.
- 10 bits are sufficient to uniquely identify any one of them.

---

## Storage Requirement

Real computers store information using whole bits.

Therefore:

bits_needed = ceil(H)

where:

ceil(x) = round x upward to the nearest integer

Example:

H = 16.29 bits

bits_needed = ceil(H) = 17 bits

This is the minimum binary storage required to uniquely encode every state.

---

## Relationship

State Space:
Ω

↓

Entropy:
H = log₂(Ω)

↓

Minimum Storage:
bits_needed = ceil(H)

State space is the fundamental quantity.

Entropy is a compressed representation of state space.

Storage is the minimum number of binary digits required to represent the state space.
