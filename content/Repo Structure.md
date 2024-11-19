---
---

#software-engineering #git

# Open problems

## Dev: Sharing resources created by Docker Compose across projects

### Examples

1. `hypotenuse/hypotenuse` creates a localstack instance, and `hypotenuse/language-model` creates its own localstack instance, with no connection.
2. `hypotenuse/loki` requires a SQS queue, which we could create in localstack. But no easy way for `hypotenuse/backend` to push to that queue for Loki to consume
3.

### Related

[[Monorepos using git subtree]]
