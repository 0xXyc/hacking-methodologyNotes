---
description: 'Challenge #1'
---

# Unstoppable

## Mission

There's a lending pool with a million DVT tokens in balance, offering flash loans for free.

If only there was a way to <mark style="color:yellow;">attack and stop the pool from offering flash loans</mark> ...

"You start with 100 DVT tokens in balance."

* The goal is to stop the pool from offering <mark style="color:yellow;">flash loans</mark>
* Essentially a Denial of Service (DoS) attack

## Exploit

* We know that we need to cause a form of DoS
* We need to discover where the flash loan function occurs

### Flash Loan Function

```
// ...
require(borrowAmount > 0, "Must borrow at least one token");
require(balanceBefore >= borrowAmount, "Not enough tokens in pool");
assert(poolBalance == balanceBefore);
require(
  balanceAfter >= balanceBefore,
  "Flash loan hasn't been paid back"
);
```

* Now we have identified where the flash loan function takes place
* The borrow amount must be greater than zero
*
