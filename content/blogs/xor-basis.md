---
title: "XOR Basis in Competitive Programming"
date: 2026-10-22T02:01:00+05:30
draft: false
description: "A practical guide to XOR bases: insertion, representability, maximum subset XOR, canonical form, and common competitive-programming patterns."
math: true
---

An XOR basis is a compact representation of every value obtainable by XORing a subset of a collection of integers. It is often introduced as “Gaussian elimination on bits,” which is correct, but not especially helpful when you first need to implement it during a contest.

The useful mental model is simpler:

> Keep at most one number for each highest set bit. When two numbers have the same highest set bit, XOR them to eliminate that bit.

That single invariant gives us insertion, membership queries, maximum subset XOR, and several more advanced operations in \(O(B)\) time, where \(B\) is the number of bits.

## Why XOR behaves like linear algebra

Treat every \(B\)-bit integer as a vector over the field \(\mathbb{F}_2\). Each bit is a coordinate, and addition is XOR because

$$
0 + 0 = 0, \qquad
0 + 1 = 1, \qquad
1 + 1 = 0.
$$

For example, the integer \(10\) is the vector \(1010_2\). Choosing a subset of numbers and XORing it is exactly the same as taking a linear combination whose coefficients are either zero or one.

An XOR basis stores a linearly independent set that spans the same space as all inserted numbers. If its rank is \(r\), then exactly \(2^r\) distinct subset XOR values are possible.

## The invariant

Let `basis[b]` be either zero or a number whose highest set bit is `b`. There can be only one basis vector for each highest set bit.

To insert a number `x`, inspect its bits from high to low:

1. Ignore a bit if it is not set in `x`.
2. If `basis[b]` is empty, store `x` there. It adds a new independent direction.
3. Otherwise, set `x ^= basis[b]`. This removes bit `b` from `x`, so elimination can continue.
4. If `x` becomes zero, it was already representable by the basis.

Here is a minimal implementation for 64-bit non-negative values:

```cpp
#include <array>
#include <cstdint>

struct XorBasis {
    static constexpr int BITS = 64;
    using U64 = std::uint64_t;

    std::array<U64, BITS> basis{};
    int rank = 0;

    // Returns true exactly when x is independent of the current basis.
    bool insert(U64 x) {
        for (int b = BITS - 1; b >= 0; --b) {
            if (((x >> b) & 1ULL) == 0) {
                continue;
            }

            if (basis[b] == 0) {
                basis[b] = x;
                ++rank;
                return true;
            }

            x ^= basis[b];
        }

        return false;
    }
};
```

The structure contains at most 64 numbers regardless of how many values are inserted.

## A small example

Insert \(10\), \(12\), and \(5\):

| Value | Binary | Elimination | Stored at |
|---:|:---:|:---|:---:|
| 10 | `1010` | empty basis | bit 3 |
| 12 | `1100` | `1100 ^ 1010 = 0110` | bit 2 |
| 5 | `0101` | `0101 ^ 0110 = 0011` | bit 1 |

The resulting basis contains `1010`, `0110`, and `0011`. Its rank is three, so it represents \(2^3 = 8\) distinct values.

Notice that the stored vectors do not have to be original input values. They only need to span the same XOR space.

## Can a target be formed?

Reduce the target using the same elimination process. If nothing remains, the target lies in the span.

```cpp
bool canMake(std::uint64_t x) const {
    for (int b = BITS - 1; b >= 0; --b) {
        if (((x >> b) & 1ULL) == 0) {
            continue;
        }

        if (basis[b] == 0) {
            return false;
        }

        x ^= basis[b];
    }

    return true;
}
```

This is also a useful way to think about insertion: `insert(x)` succeeds if and only if `canMake(x)` was false before the insertion.

## Maximum subset XOR

Start with zero and visit basis vectors from high bit to low bit. Use a vector whenever it increases the current answer.

```cpp
std::uint64_t maxXor(std::uint64_t seed = 0) const {
    std::uint64_t answer = seed;

    for (int b = BITS - 1; b >= 0; --b) {
        if ((answer ^ basis[b]) > answer) {
            answer ^= basis[b];
        }
    }

    return answer;
}
```

Why is this greedy correct? At bit `b`, only `basis[b]` can change that bit without changing a higher bit that has already been decided. A set bit at the highest position where two values differ always makes the value larger, so choosing the better option locally is permanent.

Passing a non-zero `seed` answers a slightly more general query:

$$
\max_{y \in \operatorname{span}(\text{basis})} (\text{seed} \oplus y).
$$

This form appears in tree-path and graph-cycle problems.

## A complete contest template

```cpp
#include <array>
#include <cstdint>

struct XorBasis {
    static constexpr int BITS = 64;
    using U64 = std::uint64_t;

    std::array<U64, BITS> basis{};
    int rank = 0;

    bool insert(U64 x) {
        for (int b = BITS - 1; b >= 0; --b) {
            if (((x >> b) & 1ULL) == 0) continue;
            if (basis[b] == 0) {
                basis[b] = x;
                ++rank;
                return true;
            }
            x ^= basis[b];
        }
        return false;
    }

    bool canMake(U64 x) const {
        for (int b = BITS - 1; b >= 0; --b) {
            if (((x >> b) & 1ULL) == 0) continue;
            if (basis[b] == 0) return false;
            x ^= basis[b];
        }
        return true;
    }

    U64 maxXor(U64 seed = 0) const {
        U64 answer = seed;
        for (int b = BITS - 1; b >= 0; --b) {
            if ((answer ^ basis[b]) > answer) {
                answer ^= basis[b];
            }
        }
        return answer;
    }

    void merge(const XorBasis& other) {
        for (U64 x : other.basis) {
            if (x != 0) insert(x);
        }
    }
};
```

Every operation takes \(O(B)\), except `merge`, which takes \(O(B^2)\). With \(B = 64\), both are effectively constant time in most competitive-programming constraints.

## The k-th smallest distinct subset XOR

The insertion basis is in row-echelon form, but its vectors are not necessarily ordered so that coefficient masks produce values in sorted order. For the \(k\)-th smallest query, first clear lower pivot bits from every higher basis vector.

```cpp
#include <optional>
#include <vector>

std::vector<U64> canonicalVectors() const {
    auto reduced = basis;

    for (int high = 0; high < BITS; ++high) {
        if (reduced[high] == 0) continue;

        for (int low = 0; low < high; ++low) {
            if (reduced[low] != 0 &&
                ((reduced[high] >> low) & 1ULL)) {
                reduced[high] ^= reduced[low];
            }
        }
    }

    std::vector<U64> vectors;
    for (U64 x : reduced) {
        if (x != 0) vectors.push_back(x);
    }
    return vectors;
}

// Zero-indexed; k = 0 returns the empty subset XOR, which is zero.
std::optional<U64> kthSmallest(U64 k) const {
    const auto vectors = canonicalVectors();

    if (vectors.size() < 64 && k >= (1ULL << vectors.size())) {
        return std::nullopt;
    }

    U64 answer = 0;
    for (std::size_t i = 0; i < vectors.size(); ++i) {
        if ((k >> i) & 1ULL) {
            answer ^= vectors[i];
        }
    }
    return answer;
}
```

After reduction, the pivot columns behave like an identity matrix. The highest differing coefficient between two masks determines the highest differing bit in their XOR values. Therefore, increasing coefficient masks generate distinct XOR values in increasing order.

This returns the \(k\)-th smallest **distinct value**, not the \(k\)-th subset. If \(n\) inserted numbers have rank \(r\), every distinct value is produced by \(2^{n-r}\) subsets.

## Recovering an actual subset

Sometimes the value alone is not enough; the problem asks for the indices that produced it. Store provenance beside every basis vector:

- begin an insertion with a bitset containing only the new element's index;
- whenever `x ^= basis[b]`, also XOR its provenance bitset with the provenance of `basis[b]`;
- when a new pivot is stored, store its provenance too;
- reduce a query target in the same way while accumulating provenance.

For small \(n\), `std::bitset<N>` works well. For large \(n\), explicitly storing every provenance set can dominate both time and memory, so only add it when the problem requires reconstruction.

## Where XOR bases appear

### Maximum XOR over a subset

Insert every array value and call `maxXor()`. This is the direct application.

### Cycle XORs in a graph

Assign each vertex its XOR distance from a root along a DFS tree. Every non-tree edge creates a cycle value

$$
\operatorname{dist}[u] \oplus w(u,v) \oplus \operatorname{dist}[v].
$$

Insert all such cycle values into a basis. Any walk from the root to a vertex can be modified by XORing combinations of cycles, so `maxXor(dist[v])` gives the best reachable XOR value under the problem's walk rules.

### Segment trees

Store a basis in every node. A parent basis is obtained by merging its children's bases. Range queries merge \(O(\log n)\) bases.

The merge cost is \(O(B^2)\), so the practical constant matters. If values are below \(2^{20}\), use 20 bits instead of blindly using 64.

### Offline intervals and rollback

Insertion is easy; deletion is not. If values are active over time intervals, add them to a segment tree over time and run a DFS with a rollback-capable basis. Record which pivot changed during an insertion, then restore it when leaving the node.

This pattern avoids trying to delete an arbitrary vector from a basis, an operation for which the simple representation does not contain enough information.

## Common mistakes

1. **Using signed integers.** Bit 63 makes signed comparisons and shifts awkward. Prefer `std::uint64_t`.
2. **Writing `1 << b`.** The literal `1` is usually a 32-bit signed integer. Test a bit with `(x >> b) & 1ULL` instead.
3. **Confusing input size with rank.** Dependent values do not increase the rank. The number of distinct subset XORs is \(2^{\text{rank}}\), not necessarily \(2^n\).
4. **Using the insertion basis directly for k-th queries.** First convert it to canonical form.
5. **Forgetting zero.** Zero is always representable by the empty subset. Inserting zero never increases the rank.
6. **Trying to delete normally.** Use offline intervals, rollback, or a data structure whose nodes can be rebuilt and merged.
7. **Choosing too few bits.** If values can reach \(10^{18}\), a 30-bit basis silently gives wrong answers.

## A useful checklist

When a problem combines XOR with subsets, paths, cycles, or range queries, ask:

- Is the answer determined only by which values are XORed, not by their order?
- Do I need the span of many values rather than the values themselves?
- Is the bit width small enough that \(O(B)\) or \(O(B^2)\) is cheap?
- Do I only insert, or will I need rollback or interval processing?
- Do I need one optimum, membership, the k-th distinct value, or the actual subset?

If the first three answers are yes, an XOR basis is usually the right abstraction. The main implementation is only Gaussian elimination specialized to bits; the real advantage is recognizing that many apparently different XOR problems are asking questions about the same small linear space.
