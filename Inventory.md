# Inventory State Space Model (Java Edition 1.21.11)

This document defines the formal mathematical model for the Minecraft inventory state space.

It describes how individual item slots, multi-slot inventories, recursive bundles, and shulker-box expansion contribute to the total inventory state space.

---

## Wrong page? Click here to go back:

* [Go back to Player State Space](3.Player.md)

---

# 1. Inventory State Definition

An inventory state is any configuration that can be distinguished by the game through:

* Game engine logic
* Memory representation
* NBT data
* Redstone behavior
* Comparators
* Packets
* Commands
* Any other mechanically detectable system

A distinct state exists only if the game stores it as a valid, persistent, serialized, and distinguishable state associated with the item or its container slot.

---

# 2. State-Space Framework

The primary quantity used throughout this calculation is the **state space**:

$$
\Omega = \text{number of possible states}
$$

The information content of a state space is:

$$
H = \log_2(\Omega)
$$

where $H$ is measured in bits.

The minimum number of bits required to distinguish every state exactly is:

$$
\text{bits}_{\text{needed}} = \lceil H \rceil
$$

The corresponding whole-byte storage requirement is:

$$
S = \left\lceil\frac{H}{8}\right\rceil
$$

The hierarchy is therefore:

1. **State Space**

   * $\Omega$
2. **Information Content**

   * $H = \log_2(\Omega)$
3. **Minimum Binary Representation**

   * $\lceil H\rceil$ bits
4. **Whole-Byte Storage**

   * $\left\lceil H/8\right\rceil$ bytes

The state space $\Omega$ is the primary mathematical object. Entropy and storage are derived quantities.

---

# 3. Single Slot Models

## 3.1 One Slot, One Item Type

For an item with a maximum stack size of 64, a slot can contain:

* Empty
* 1 item
* 2 items
* ...
* 64 items

Therefore:

$$
\Omega = 65
$$

and:

$$
H = \log_2(65)
\approx 6.022\text{ bits}
$$

The minimum exact binary representation is:

$$
\text{bits}_{\text{needed}} = 7
$$

---

## 3.2 One Slot, 1,238 Stack-64 Item Types

For the 1,238 item types that stack to 64, each item type has 64 non-empty quantities.

Including the empty state:

$$
\Omega =
1238\times64
+
26\times16
+
225\times1
+
1
$$

which gives:

$$
\boxed{\Omega_{\text{slot}}=79\,874}
$$

before recursive bundle states are substituted into the model.

The corresponding information content is:

$$
H_{\text{slot}}=\log_2(79\,874)\approx16.29\text{ bits}
$$

and therefore:

$$
\text{bits}_{\text{needed}}=17
$$

---

# 4. Multi-Slot Inventory

## 4.1 General Rule

For $n$ independent slots, each with $\Omega_{\text{slot}}$ possible states:

$$
\boxed{\Omega_{\text{total}}=\Omega_{\text{slot}}^n}
$$

The corresponding information content is:

$$
\boxed{H_{\text{total}}=n\log_2(\Omega_{\text{slot}})}
$$

---

## 4.2 Two Slots

For two independent slots:

$$
\Omega_{\text{total}}=79\,874^2
$$

$$
\Omega_{\text{total}}=6\,379\,859\,876
$$

and:

$$
H_{\text{total}}\approx32.57\text{ bits}
$$

Thus:

$$
\text{bits}_{\text{needed}}=33
$$

---

## 4.3 Five Slots

For five independent slots:

$$
\Omega_{\text{total}}=79\,874^5
$$

with:

$$
H_{\text{total}}\approx81.43\text{ bits}
$$

Therefore:

$$
\text{bits}_{\text{needed}}=82
$$

---

# 5. Bundle State Space

Bundles are substantially more complicated than ordinary inventory slots because all of their contents compete for a shared capacity of 64 units.

## 5.1 Bundle Capacity

A bundle has:

$$
\boxed{C=64}
$$

The capacity cost of an item depends on its maximum stack size:

| Maximum stack size | Capacity cost |
| -----------------: | ------------: |
|                 64 |             1 |
|                 16 |             4 |
|                  1 |            64 |

Thus:

* A stack-64 item uses 1 capacity unit per item.
* A stack-16 item uses 4 capacity units per item.
* A stack-size-1 item uses 64 capacity units.

The calculation uses:

* 1,238 stack-64 item types
* 26 stack-16 item types
* 225 stack-size-1 item types

---

## 5.2 Base Bundle Model

For a stack-64 item type, the generating-function contribution is:

$$
1+t+t^2+\cdots+t^{64}
$$

For a stack-16 item type:

$$
1+t^4+t^8+\cdots+t^{64}
$$

For a stack-size-1 item type:

$$
1+t^{64}
$$

The initial non-recursive generating function is therefore:

$$
G_0(t)=(1+t+\cdots+t^{64})^{1238}(1+t^4+t^8+\cdots+t^{64})^{26}(1+t^{64})^{225}
$$

The coefficient of $t^n$ counts configurations using exactly $n$ capacity units.

Thus:

$$
\Omega_0=\sum_{n=0}^{64}[t^n]G_0(t)
$$

The calculated result was approximately:

$$
\log_{10}(\Omega_0)\approx109.548083539009
$$

or:

$$
H_0\approx363.910856\text{ bits}
$$

This model, however, does not yet account correctly for recursive bundles.

---

# 6. Corrected Recursive Bundle Model

## 6.1 Why the First Recursive Model Was Incorrect

An earlier model treated every recursive bundle state as though it were an ordinary stack-64 item.

This led to the approximation:

$$
\Omega_d\approx\frac{\Omega_{d-1}^{64}}{64!}
$$

and ultimately produced the obsolete estimate:

$$
\Omega_{64}\approx2^{1.37\times10^{118}}
$$

This was incorrect.

The problem was that a nested bundle does **not** always occupy one capacity unit.

---

## 6.2 Actual Nested-Bundle Capacity Cost

A nested bundle containing contents that use $k$ capacity units costs:

$$
\boxed{k+4}
$$

capacity units when placed inside its parent bundle.

An empty nested bundle therefore costs:

$$
4
$$

capacity units.

Consequently, the maximum nesting depth is:

$$
\boxed{d_{\max}=\left\lfloor\frac{64}{4}\right\rfloor=16}
$$

The maximum bundle nesting depth is therefore **16**, not 64.

---

## 6.3 Bundle Colour Variants

Minecraft has 17 distinguishable bundle variants:

* 1 normal bundle
* 16 coloured bundles

The original 225 stack-size-1 item types therefore consist of:

* 208 ordinary stack-size-1 item types
* 17 bundle variants

The bundle variants must be treated separately because they can recursively contain other bundles.

---

## 6.4 Capacity Distribution

A single total count of bundle states is insufficient because recursive bundles can have different capacity costs.

Define:

$$
C_{d,k}
$$

as the number of bundle-content states with maximum nesting depth at most $d$ that use exactly $k$ capacity units.

The total number of bundle-content states at depth $d$ is:

$$
\boxed{C_d=\sum_{k=0}^{64}C_{d,k}}
$$

This capacity distribution is required because a nested bundle containing $k$ units of content costs $k+4$ units in its parent.

---

## 6.5 Corrected Base Case

At depth $d=0$, bundles cannot contain other bundles.

The ordinary stack-64 items contribute:

$$
(1+t+\cdots+t^{64})^{1238}
$$

The 26 stack-16 items contribute:

$$
(1+t^4+t^8+\cdots+t^{64})^{26}
$$

The 208 ordinary stack-size-1 items contribute:

$$
(1+t^{64})^{208}
$$

The 17 bundle variants are empty bundles at this level, each costing 4 capacity units:

$$
(1+t^4+t^8+\cdots+t^{64})^{17}
$$

Combining the two groups with a capacity cost of 4 gives:

$$
\boxed{G_0(t)=(1+t+\cdots+t^{64})^{1238}(1+t^4+t^8+\cdots+t^{64})^{43}(1+t^{64})^{208}}
$$

Therefore:

$$
C_{0,k}=[t^k]G_0(t)
$$

and:

$$
C_0=\sum_{k=0}^{64}C_{0,k}
$$

---

## 6.6 Recursive Step

Suppose a bundle at depth $d$ contains a recursive bundle whose contents use $j$ capacity units.

That nested bundle costs:

$$
j+4
$$

capacity units in its parent.

For each content state, there are 17 possible bundle colours. Therefore, the number of distinct coloured recursive bundle states with content cost $j$ is:

$$
17C_{d-1,j}
$$

Multiple copies of the same exact recursive bundle state are allowed.

For $m$ copies chosen from $N$ distinct states, the number of possible multisets is:

$$
\binom{N+m-1}{m}
$$

where:

$$
N=17C_{d-1,j}
$$

Therefore, recursive bundle states with content cost $j$ contribute:

$$
\binom{17C_{d-1,j}+m-1}{m}
$$

states at capacity:

$$
m(j+4)
$$

Only:

$$
j\le60
$$

can contribute, because every nested bundle requires at least 4 additional capacity units.

The recursion therefore evolves the complete vector:

$$
\boxed{(C_{d,0},C_{d,1},\ldots,C_{d,64})}
$$

rather than a single scalar state count.

---

# 7. Bundle Recursion and Convergence

The corrected calculation was performed in logarithmic space because the state counts become extremely large.

The calculated total state spaces were:

$$
\log_{10}(C_0)\approx109.5481228956097
$$

$$
\log_{10}(C_1)\approx127.6379710569422
$$

$$
\log_{10}(C_2)\approx128.4111439133446
$$

$$
\log_{10}(C_3)\approx128.5656468629944
$$

$$
\log_{10}(C_4)\approx128.5666089521589
$$

$$
\log_{10}(C_5)\approx128.5666091017208
$$

$$
\log_{10}(C_6)\approx128.5666091017222
$$

The increase rapidly becomes extremely small.

From depth 4 to depth 5, the relative increase is approximately:

$$
3.44\times10^{-5}\%
$$

From depth 5 to depth 6:

$$
3.14\times10^{-10}\%
$$

Further changes become smaller than the numerical precision used by the calculation.

This does **not** mean that the mathematical changes are exactly zero. It means that they are below the numerical resolution of the calculation.

The corrected result therefore converges to:

$$
\boxed{\log_{10}(C_{16})\approx128.5666091017222}
$$

and:

$$
\boxed{\log_2(C_{16})\approx427.0890308394121\text{ bits}}
$$

Here, $C_{16}$ represents the state space of the **contents** of a bundle, including all valid recursive configurations up to the maximum nesting depth of 16.

---

# 8. Complete Bundle Item State Space

$C_{16}$ describes bundle contents only.

The bundle item itself has 17 distinguishable colour variants.

Therefore:

$$
\boxed{\Omega_{\text{bundle}}=17C_{16}}
$$

Numerically:

$$
\boxed{\log_{10}(\Omega_{\text{bundle}})\approx129.7970580231005}
$$

and:

$$
\boxed{H_{\text{bundle}}\approx431.1764936806624\text{ bits}}
$$

The minimum exact binary representation is therefore:

$$
\boxed{432\text{ bits}}
$$

or:

$$
\boxed{54\text{ bytes}}
$$

It is important to distinguish:

* $C_{16}$ — state space of bundle **contents**
* $\Omega_{\text{bundle}}$ — state space of the complete **coloured bundle item**

---

# 9. Corrected Single-Slot State Space

Before recursive bundles were included, one inventory slot contained:

$$
79\,874
$$

possible states.

These states already included the 17 ordinary bundle variants.

Those 17 states must therefore be removed before inserting the complete recursive bundle state space.

Thus:

$$
\boxed{\Omega_{\text{slot}}=79\,874-17+17C_{16}}
$$

or equivalently:

$$
\boxed{\Omega_{\text{slot}}=79\,874+17(C_{16}-1)}
$$

The subtraction prevents the original 17 bundle states from being counted twice.

Since the recursive bundle contribution dominates the ordinary item states:

$$
\Omega_{\text{slot}}\approx17C_{16}
$$

giving:

$$
\boxed{\log_{10}(\Omega_{\text{slot}})\approx129.7970580231005}
$$

and:

$$
\boxed{\log_2(\Omega_{\text{slot}})\approx431.1764936806624\text{ bits}}
$$

---

# 10. Inventory Topology and Shulker Expansion

The inventory model includes the cursor as an independent top-level slot.

The top-level slots are:

* 36 inventory slots
* 1 off-hand slot
* 4 crafting slots
* 4 armor slots
* 1 cursor slot

Therefore:

$$
\boxed{46\text{ top-level slots}}
$$

A shulker box occupies one slot while providing 27 internal slots.

Its net contribution is therefore:

$$
27-1=26
$$

effective slots.

If $k$ top-level shulker boxes are present:

$$
\boxed{S(k)=46+26k}
$$

with:

$$
0\le k\le46
$$

The maximum number of top-level shulker boxes is therefore 46, giving:

$$
S_{\max}=46+26(46)=\boxed{1242}
$$

effective slots.

---

# 11. Complete Inventory State Space

For a fixed topology containing $k$ top-level shulker boxes:

$$
\Omega(k)=\Omega_{\text{slot}}^{46+26k}
$$

The complete inventory state space is the sum over all possible top-level shulker-box topologies:

$$
\boxed{\Omega_{\text{inventory}}=\sum_{k=0}^{46}\Omega_{\text{slot}}^{46+26k}}
$$

The largest topology is:

$$
\boxed{\Omega_{\max}=\Omega_{\text{slot}}^{1242}}
$$

---

# 12. Why the Largest Topology Dominates

The terms form a geometric sequence:

$$
\Omega_{\text{slot}}^{1242},\Omega_{\text{slot}}^{1216},\Omega_{\text{slot}}^{1190},\ldots
$$

The ratio between consecutive terms is:

$$
r=\Omega_{\text{slot}}^{-26}
$$

Using:

$$
\log_{10}(\Omega_{\text{slot}})\approx129.7970580231005
$$

gives:

$$
\log_{10}(r)=-26(129.7970580231005)\approx-3374.723508600613
$$

Therefore:

$$
\boxed{r\approx1.89\times10^{-3375}}
$$

The entire remainder of the sum beyond the largest term is therefore only approximately:

$$
1.89\times10^{-3375}
$$

of the largest term.

In percentage form:

$$
\boxed{1.89\times10^{-3373}\%}
$$

The exact finite geometric sum can also be written as:

$$
\Omega_{\text{inventory}}=\Omega_{\max}\frac{1-r^{47}}{1-r}
$$

where:

$$
r=\Omega_{\text{slot}}^{-26}
$$

Because $r$ is so small, the complete inventory state space is indistinguishable from its largest topology at any practical numerical precision:

$$
\boxed{\Omega_{\text{inventory}}\approx\Omega_{\text{slot}}^{1242}}
$$

---

# 13. Final Inventory Calculation

Using:

$$
\log_{10}(\Omega_{\text{slot}})\approx129.7970580231005
$$

the largest topology has:

$$
\log_{10}(\Omega_{\max})=1242\log_{10}(\Omega_{\text{slot}})
$$

$$
\boxed{\log_{10}(\Omega_{\max})\approx161207.9460646908}
$$

Therefore:

$$
\boxed{\Omega_{\text{inventory}}\approx10^{161207.9460646908}}
$$

The corresponding information content is:

$$
H_{\text{inventory}}=1242\log_2(\Omega_{\text{slot}})
$$

giving:

$$
\boxed{H_{\text{inventory}}\approx535521.205\text{ bits}}
$$

The minimum exact binary width is:

$$
\boxed{\text{bits}_{\text{needed}}=535522\text{ bits}}
$$

For whole-byte storage:

$$
S_{\text{inventory}}=\left\lceil\frac{535521.205}{8}\right\rceil
$$

so:

$$
\boxed{S_{\text{inventory}}=66941\text{ bytes}}
$$

or approximately:

$$
\boxed{66941\text{ bytes}\approx66.941\text{ KB}\approx65.372\text{ KiB}}
$$

---

# 14. Final Result

The corrected inventory state space for the model is therefore:

$$
\boxed{\begin{aligned}\Omega_{\text{inventory}}&\approx10^{161207.946}\\H_{\text{inventory}}&\approx535521.205\text{ bits}\\\text{bits}_{\text{needed}}&=535522\text{ bits}\\S_{\text{inventory}}&=66941\text{ bytes}\\&\approx65.37\text{ KiB}\end{aligned}}
$$

---

# 15. Important Corrections

The current result differs substantially from the earlier model.

The most important corrections were:

1. **The cursor slot was included.**

   * Top-level slots increased from 45 to 46.

2. **The maximum shulker topology was corrected.**

   * 46 top-level shulkers are possible.
   * Maximum effective slot count is 1242.

3. **Bundle recursion was corrected.**

   * A nested bundle costs 4 capacity units plus the capacity used by its contents.
   * Maximum bundle nesting depth is 16 rather than 64.

4. **Bundle colours were included explicitly.**

   * There are 17 distinguishable bundle variants.

5. **The recursive calculation tracks capacity distributions.**

   * The state vector $C_{d,k}$ is required rather than a single scalar recurrence.

6. **The obsolete estimate**

   $$
   2^{1.37\times10^{118}}
   $$

   is no longer valid.

The corrected calculation instead produces:

$$
\boxed{\Omega_{\text{inventory}}\approx10^{161207.946}}
$$

with:

$$
\boxed{H_{\text{inventory}}\approx535521.205\text{ bits}}
$$

and:

$$
\boxed{S_{\text{inventory}}=66941\text{ bytes}}
$$

This is the current inventory result used by the Minecraft State-Space project.
