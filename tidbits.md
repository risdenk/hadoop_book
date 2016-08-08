# Tidbits
Some random tidbits of information captured along the way per components in a Hadoop stack.

## Hive
* Use `beeline` if you want authorization to take place via Ranger/Sentry
  * `hive` cli avoids all authorization checks

## Storm
* Always anchor tuples when emitting to get accurate counts
* Check that `topology.max.spout.pending` is set properly for your topology
* Make sure that `topology.users/groups` and/or `logs.users/groups` are set per topology