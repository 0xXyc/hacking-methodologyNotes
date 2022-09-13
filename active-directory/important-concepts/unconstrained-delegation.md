---
description: Windows AD feature abuse
---

# Unconstrained Delegation

## Impact

If a domain admin authenticates inside an endpoint with "Unconstrained Delegation" activated and you possess local admin privileges in that machine, you can dump the ticket and impersonate the Domain Admin.

## How it works

This is a feature that consists of a Domain Administrator being able to set any computer insider&#x20;
