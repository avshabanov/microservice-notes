Request ID-based Durability in Distributed System
=================================================

# The Problem

In SOA world with multiple remote services calling each other it is not uncommon that certain
services might fail.

The question is how to handle failures in complex workflows when multiple calls are involved: let's say we need to call service C that calls service A and B. Let's say call to service B fails and it resulted in failed call to service C. The question is how to retry call to service C without calling service A (which might be expensive or cause undesirable side effects if called more than once).

# The Solution

Introduce and track unique request IDs, so that every non-idempotent call will take such request ID and will do nothing if request with such request ID has been processed already.

For example:

Method ``Z foo(X x, Y y)`` can be modified as ``Z foo(ID requestId, X x, Y y)``.








