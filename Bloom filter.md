A **Bloom filter** is a **space-efficient probabilistic data structure** used to test **whether an element is possibly in a set or definitely not in a set**.

### In one sentence

> Bloom filters can tell you **“maybe yes” or “definitely no”**, but **never a guaranteed yes**.

---

## How it works (intuition)

A Bloom filter consists of:

1. A **bit array** of size `m`, initially all `0`
2. **k independent hash functions**

### Adding an element

* Hash the element using all `k` hash functions
* Each hash maps to a position in the bit array
* Set those positions to `1`

### Checking an element

* Hash it using the same `k` hash functions
* If **any** of the bits is `0` → **definitely not present**
* If **all** bits are `1` → **possibly present**

---

## Key properties

### ✅ Pros

* Extremely **memory efficient**
* **Fast** insert and lookup (`O(k)`)
* Perfect for large-scale systems

### ❌ Cons

* **False positives** are possible
* **No false negatives**
* Cannot delete elements (without variants)
* Cannot retrieve the stored elements

---

## False positives explained

Two different elements may set the **same bits**, so the filter might think an element exists when it doesn’t.

Example:

```
Alice → sets bits [3, 7, 10]
Bob   → sets bits [3, 7, 10]
```

Now querying Bob returns "maybe", even if Bob was never added.

---

## Time & Space

* **Insert:** `O(k)`
* **Query:** `O(k)`
* **Space:** `m` bits (very small compared to storing keys)

False positive rate:
[
(1 - e^{-kn/m})^k
]

Where:

* `n` = number of elements
* `m` = number of bits
* `k` = number of hash functions

---

## Common real-world uses

### 🚀 System design

* **Cache penetration prevention** (Redis, CDN)
* **Database existence checks**
* **Web crawler URL deduplication**
* **Spam filtering**
* **Distributed systems (HBase, Cassandra)**

### Example: Redis cache

```text
Request → Bloom Filter → Cache → Database
```

If Bloom filter says “not present” → skip DB entirely

---

## Variants you should know

* **Counting Bloom Filter** → supports deletion
* **Scalable Bloom Filter** → grows dynamically
* **Cuckoo Filter** → lower false positives + delete support

---

## Simple Go example

```go
type Bloom struct {
    bits []bool
}

func hash1(s string) int { return len(s) % 100 }
func hash2(s string) int { return (len(s) * 7) % 100 }

func (b *Bloom) Add(s string) {
    b.bits[hash1(s)] = true
    b.bits[hash2(s)] = true
}

func (b *Bloom) Exists(s string) bool {
    return b.bits[hash1(s)] && b.bits[hash2(s)]
}
```

---

## When should you use it?

Use a Bloom filter when:

* You **only need membership checking**
* **Memory is limited**
* **False positives are acceptable**
* **False negatives are NOT acceptable**

If you want, I can also explain:

* Bloom filters in **Redis**
* How to **size Bloom filters correctly**
* Bloom vs **LRU / Set / Cuckoo filter**
* A **real system design example**

Just tell me 👍
