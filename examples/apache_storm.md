# Apache Storm

## Topology Permissions
By default, only the user who deploys the topology has access to admin operations such as rebalance, activate, deactivate, and kill.

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

## Test Spout
```java
import backtype.storm.spout.SpoutOutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.IRichSpout;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Values;

import java.util.Map;

class MySpout implements IRichSpout {
  private int i = 0;
  private Map conf;
  private TopologyContext context;
  private SpoutOutputCollector collector;

  @Override
  public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
    this.conf = conf;
    this.context = context;
    this.collector = collector;
  }

  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("key", "message"));
  }

  @Override
  public void close() {}

  @Override
  public void activate() {}

  @Override
  public void deactivate() {}

  @Override
  public void nextTuple() {
    if(this.i < 5) {
      this.collector.emit(new Values("key" + i, "val" + i), i);
      this.i++;
    }
  }

  @Override
  public void ack(Object o) {}

  @Override
  public void fail(Object o) {}

  @Override
  public Map<String, Object> getComponentConfiguration() {
    return null;
  }
}
```