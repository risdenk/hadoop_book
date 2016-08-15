# Apache Storm

## Viewing Logs
Storm has authorization related to viewing logs. The log viewers are configured per topology instead of at the cluster level.

By default, only the user who deploy the topology have access to admin operations such as rebalance, activate, deactivate, and kill.

The following properties can be set in the topology configuration:

| Configuration | Description |
| -- | -- |
| `topology.users/groups` | This allows the users/groups specified to act as owners of the topology. This allows users to perform topology admin operations such as rebalance, activate, deactivate, and kill. |
| `logs.users/groups` | This allows the users/groups specified to look at the logs of the topology. |

The configurations above can also be set at the cluster level. They will be the default configuration if the topology does not override it.

There is one cluster level configuration that will enable a set of users to be admins for the whole Storm cluster.

| Configuration | Description |
| -- | -- |
| `nimbus.admins` | These users will have super user permissions on all topologies deployed. They will be able to perform other admin operations (such as rebalance, activate, deactivate and kill), even if they are not the owners of the topology. |