# Inventory State Space

[← Back to Player](3.Player.md)

The inventory is currently the most developed component of the player state space.

It contains several different types of slots, including ordinary inventory slots, the off-hand slot, armor slots, crafting slots, and the cursor. Some items can also contain other items, which makes the calculation significantly more complicated than simply counting the possible contents of each slot.

---

## 1. Inventory Model

The inventory is modeled as a collection of independent slots.

For a normal item slot, the possible states depend on:

* which item is present;
* how many copies of that item are present;
* whether the slot is empty;
* whether the item has additional state.

For the current calculation, the item types are distinguished according to the project's **distinguishability rule**:

> If two items can be distinguished by the game state, they are treated as different states.

The current item distribution is:

| Maximum stack size | Number of item types |
| -----------------: | -------------------: |
|                 64 |                1,238 |
|                 16 |                   26 |
|                  1 |                  225 |
|          **Total** |            **1,489** |

Items with a maximum stack size of 1 are therefore treated as individual item types rather than as stackable quantities.

---

# 2. Single-Slot State Space

## 2.1 Ordinary Items

For an item that stacks to 64, a slot can contain any quantity from 1 through 64.

Therefore, each stack-64 item contributes 64 possible non-empty states.

Similarly:

* a stack-16 item contributes 16 states;
* a stack-1 item contributes 1 state.

Including the empty slot, the total number of ordinary slot states is:

$$
\Omega_{\text{slot}}
=
1
+
1238(64)
+
26(16)
+
225
$$

which gives:

$$
\Omega_{\text{slot}}
=
79\,874
$$

The corresponding information content is:

$$
H_{\text{slot}}
=
\log_2(79\,874)
\approx
16.29\text{ bits}
$$

Since a binary representation must contain a whole number of bits, the minimum fixed-width representation for one ordinary slot would require:

$$
\left\lceil H_{\text{slot}} \right\rceil
=
17\text{ bits}
$$

---

## 2.2 Multiple Ordinary Slots

Before accounting for the cursor and shulker-box expansion, the standard player inventory model contains 45 slots:

* 36 inventory slots;
* 1 off-hand slot;
* 4 armor slots;
* 4 crafting slots.

Thus:

$$
\Omega_{45}
=
79\,874^{45}
$$

Numerically:

$$
\Omega_{45}
\approx
4.04555\times10^{220}
$$

The corresponding information content is:

$$
H_{45}
=
45\log_2(79\,874)
\approx
732.845\text{ bits}
$$

Therefore:

$$
\left\lceil H_{45}\right\rceil
=
733\text{ bits}
$$

and the equivalent whole-byte storage requirement is:

$$
\left\lceil\frac{733}{8}\right\rceil
=
92\text{ bytes}
$$

This is only the starting point, because the cursor and containers such as shulker boxes expand the effective state space considerably.

---

# 3. Bundle State Space

Bundles are significantly more complicated than ordinary stackable items because their contents can themselves consist of other items.

The bundle has a capacity of 64 units.

The capacity cost of an item depends on its maximum stack size:

| Maximum stack size | Capacity cost |
| -----------------: | ------------: |
|                 64 |             1 |
|                 16 |             4 |
|                  1 |            64 |

Thus, a bundle cannot simply be treated as another ordinary stack-64 item.

---

## 3.1 First Non-Recursive Model

The first generating function models the contents of a bundle without allowing bundles to contain other bundles.

For the 1,238 stack-64 item types, the possible quantities are represented by:

$$
1+t+t^2+\dots+t^{64}
$$

For the 26 stack-16 item types:

$$
1+t^4+t^8+\dots+t^{64}
$$

For the 225 stack-1 item types:

$$
1+t^{64}
$$

The resulting generating function is therefore:

$$
G_0(t)
=
(1+t+t^2+\dots+t^{64})^{1238}
(1+t^4+t^8+\dots+t^{64})^{26}
(1+t^{64})^{225}
$$

The number of possible bundle-content states is the sum of the coefficients up to capacity 64:

$$
C_0
=
\sum_{n=0}^{64}
[t^n]G_0(t)
$$

This gives approximately:

$$
\log_{10}(C_0)
\approx
109.5481228956097
$$

or:

$$
\log_2(C_0)
\approx
363.91\text{ bits}
$$

---

# 4. Recursive Bundles

The previous model is incomplete because bundles can contain bundles.

A recursive bundle consumes:

$$
\text{contents capacity}+4
$$

capacity units.

The additional 4 units represent the bundle itself.

Therefore, an empty bundle already costs 4 capacity units.

Since the total capacity is 64:

$$
\left\lfloor\frac{64}{4}\right\rfloor
=
16
$$

Thus, the maximum possible nesting depth is **16**.

---

## 4.1 Bundle Variants

There are 17 bundle variants:

* 1 normal bundle;
* 16 colored bundles.

These variants must be treated as distinct item types.

The 225 stack-1 item types can therefore be separated into:

* 208 ordinary stack-1 item types;
* 17 bundle variants.

The base generating function can consequently be written as:

$$
G_0(t)
=
(1+t+t^2+\dots+t^{64})^{1238}
(1+t^4+t^8+\dots+t^{64})^{43}
(1+t^{64})^{208}
$$

The exponent 43 in the second term consists of:

$$
26+17=43
$$

representing the 26 stack-16 items and the 17 empty bundle variants.

---

## 4.2 Recursive State Counting

Let:

$$
C_{d,k}
$$

be the number of possible bundle-content states using exactly \(k\) capacity units, with recursive bundle nesting allowed to depth \(d\).

The total number of states at depth \(d\) is:

$$
C_d
=
\sum_{k=0}^{64}C_{d,k}
$$

A recursive bundle whose contents use \(j\) capacity units consumes:

$$
j+4
$$

capacity units in its containing bundle.

For each possible content state there are 17 colored bundle variants.

Therefore, if there are:

$$
N=17C_{d-1,j}
$$

possible recursive bundle items with content capacity \(j\), then allowing multiple copies gives the multiset count:

$$
\binom{N+m-1}{m}
$$

for \(m\) copies.

Only values satisfying:

$$
j\leq60
$$

can contribute, because the bundle itself requires another 4 capacity units.

This recurrence is evaluated across the complete capacity vector:

$$
(C_{d,0},C_{d,1},\dots,C_{d,64})
$$

rather than treating the total number of states as a single scalar.

---

# 5. Convergence

The recursive calculation converges rapidly.

The calculated values are:

| Maximum depth | \(\log_{10}(C_d)\) |
| ------------: | -----------------: |
|             0 |  109.5481228956097 |
|             1 |  127.6379710569422 |
|             2 |  128.4111439133446 |
|             3 |  128.5656468629944 |
|             4 |  128.5666089521589 |
|             5 |  128.5666091017208 |
|             6 |  128.5666091017222 |

The changes become extremely small after depth 4.

From depth 5 to depth 6, the relative change is only approximately:

$$
3.14\times10^{-10}\%
$$

At depth 6, the result has stabilized to the displayed numerical precision.

Therefore, the final bundle-content state count is:

$$
\log_{10}(C_{16})
\approx
128.5666091017222
$$

and:

$$
\log_2(C_{16})
\approx
427.0890308394121\text{ bits}
$$

The recursion only needs to reach depth 16 in principle; in practice, the numerical result has already converged much earlier.

---

# 6. Complete Bundle State Space

A complete bundle also has 17 possible color variants.

Therefore:

$$
\Omega_{\text{bundle}}
=
17C_{16}
$$

giving:

$$
\log_{10}(\Omega_{\text{bundle}})
\approx
129.7970580231005
$$

and:

$$
H_{\text{bundle}}
=
\log_2(\Omega_{\text{bundle}})
\approx
431.1764936806624\text{ bits}
$$

The minimum fixed-width binary representation therefore requires:

$$
\left\lceil
431.1764936806624
\right\rceil
=
432\text{ bits}
$$

or:

$$
\frac{432}{8}
=
54\text{ bytes}
$$

---

# 7. Corrected Single-Slot State Space

The ordinary slot calculation already included 17 stack-1 bundle entries implicitly.

However, those entries only represented the empty state of each bundle variant.

The recursive bundle calculation replaces those entries with the complete bundle state space.

Therefore, the corrected slot state space is:

$$
\Omega_{\text{slot}}
=
79\,874
-
17
+
17C_{16}
$$

which gives:

$$
\Omega_{\text{slot}}
=
79\,874
-
17
+
17C_{16}
$$

Numerically:

$$
\log_{10}(\Omega_{\text{slot}})
\approx
129.7970580231005
$$

and:

$$
H_{\text{slot}}
=
\log_2(\Omega_{\text{slot}})
\approx
431.1764936806624\text{ bits}
$$

Thus, one corrected inventory slot requires:

$$
\left\lceil
431.1764936806624
\right\rceil
=
432\text{ bits}
$$

or:

$$
54\text{ bytes}
$$

for a fixed-width representation.

---

# 8. Cursor Correction

The cursor is an additional independent location capable of holding an item stack.

Therefore, the total number of top-level inventory slots is:

$$
46
$$

rather than 45.

This distinction is important because shulker-box expansion is based on the number of top-level slots that can themselves contain shulker boxes.

---

# 9. Shulker-Box Expansion

A shulker box occupies one ordinary slot but contains 27 additional inventory slots.

Therefore, each shulker box provides a net increase of:

$$
27-1=26
$$

effective slots.

If \(k\) shulker boxes are present in the top-level inventory, the effective number of slots is:

$$
S(k)
=
46+26k
$$

The maximum number of top-level shulker boxes is 46, giving:

$$
S_{\max}
=
46+26(46)
=
1242
$$

Thus, the maximum effective inventory size is:

$$
1242\text{ slots}
$$

---

## 9.1 Full Inventory State Space

The complete inventory state space is a sum over all possible numbers of top-level shulker boxes:

$$
\Omega_{\text{inventory}}
=
\sum_{k=0}^{46}
\Omega_{\text{slot}}^{46+26k}
$$

The largest term corresponds to \(k=46\):

$$
\Omega_{\max}
=
\Omega_{\text{slot}}^{1242}
$$

Because the corrected single-slot state space is enormous, this final topology overwhelmingly dominates the sum.

The ratio between consecutive terms is:

$$
r
=
\Omega_{\text{slot}}^{-26}
$$

Using:

$$
\log_{10}(\Omega_{\text{slot}})
\approx
129.7970580231005
$$

gives:

$$
\log_{10}(r)
\approx
-3374.723508600613
$$

Therefore:

$$
r
\approx
1.89\times10^{-3375}
$$

The contribution of all smaller topologies is consequently negligible compared with the maximum topology.

The finite sum can also be written as:

$$
\Omega_{\text{inventory}}
=
\Omega_{\max}
\frac{1-r^{47}}{1-r}
$$

---

# 10. Final Inventory State Space

The dominant topology contains 1,242 effective slots.

Therefore:

$$
\log_{10}(\Omega_{\max})
=
1242
\log_{10}(\Omega_{\text{slot}})
$$

which gives:

$$
\log_{10}(\Omega_{\max})
\approx
161207.9460646908
$$

The complete inventory state space is therefore approximately:

$$
\boxed{
\Omega_{\text{inventory}}
\approx
10^{161207.9460646908}
}
$$

The corresponding information content is:

$$
H_{\text{inventory}}
=
1242
\log_2(\Omega_{\text{slot}})
$$

giving:

$$
\boxed{
H_{\text{inventory}}
\approx
535521.205\text{ bits}
}
$$

The minimum exact fixed-width binary representation therefore requires:

$$
\boxed{
535522\text{ bits}
}
$$

Since storage is measured in whole bytes:

$$
\left\lceil
\frac{535522}{8}
\right\rceil
=
66941\text{ bytes}
$$

Therefore:

$$
\boxed{
S_{\text{inventory}}
=
66941\text{ bytes}
}
$$

or approximately:

$$
\boxed{
66.941\text{ KB}
}
$$

using decimal kilobytes, or:

$$
\boxed{
65.372\text{ KiB}
}
$$

using binary kibibytes.

---

# 11. Summary

The corrected inventory calculation produces:

| Quantity                          |                        Result |
| --------------------------------- | ----------------------------: |
| Corrected single-slot state space |    \(\approx10^{129.797058}\) |
| Single-slot information           |    \(\approx431.176494\) bits |
| Maximum effective slots           |                         1,242 |
| Inventory state space             | \(\approx10^{161207.946065}\) |
| Inventory information             |   \(\approx535,521.205\) bits |
| Minimum exact binary width        |                  535,522 bits |
| Storage                           |                  66,941 bytes |
| Decimal storage                   |                     66.941 KB |
| Binary storage                    |                    65.372 KiB |

The inventory is therefore already an extraordinarily large contributor to the total Minecraft state space.

The calculation also demonstrates why nested containers cannot simply be treated as ordinary items: their internal states recursively increase the number of distinguishable states available in every containing inventory.

---

## 12. Important Assumptions

This calculation depends on the following assumptions:

1. The item distribution is based on the current project data for Minecraft Java Edition 1.21.11.
2. Distinguishable item types are treated as separate states.
3. The cursor is an independent top-level inventory location.
4. Shulker boxes provide 27 internal slots while consuming one containing slot.
5. Bundle capacity is 64 units.
6. Bundle contents consume capacity according to their maximum stack size.
7. Bundles can contain other bundles.
8. There are 17 distinct bundle variants.
9. Bundle nesting has a theoretical maximum depth of 16.
10. The recursive bundle calculation counts distinct bundle contents according to the multiset model described above.
11. The inventory calculation sums over all possible numbers of top-level shulker boxes.
12. The largest shulker topology dominates the total inventory state space to an overwhelmingly large degree.

These assumptions are part of the mathematical model and may be revised if Minecraft mechanics or the project's definition of a distinguishable state changes.

---

[← Back to Player](3.Player.md)
