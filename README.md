# A Docker image for Apache Flink (1.1.2)

A base image for creating Apache Flink clusters. Usable to create jobmanagers or taskmanagers.

## Usage

### Configuration properties

You can set every [https://ci.apache.org/projects/flink/flink-docs-release-1.1/setup/config.html](config property) by passing it as lowercase environment variable in the Marathon app definition's `env` map, where the variable name is the original property with `_` instead of `.`, prepended by `flink_`.

For example, if you'd want to set `recovery.mode`, you'd have to specify `flink_recovery_mode` as environment variable (e.g. `-e flink_recovery_mode=zookeeper`) with an appropriate value.

Those environment variables will be automatically added to the `flink-config.yaml` file by the `docker-entrypoint.sh` script before the startup.

## Running

### Via standalone Docker

Start a standalone JobManager (with host networking, binding on 127.0.0.1):

```
docker run -d \
  --name JobManager \
  --net=host \
  -e HOST=127.0.0.1 \
  -e PORT0=6123 \
  -e PORT1=8081 \
  mesoshq/flink:1.1.2 jobmanager
```

Start a TaskManager:

```
docker run -d \
  --name TaskManager \
  --net=host \
  -e flink_jobmanager_rpc_address=127.0.0.1 \
  -e flink_jobmanager_rpc_port=6123 \
  -e flink_taskmanager_tmp_dirs=/data/tasks \
  -e flink_blob_storage_directory=/data/blobs \
  -e flink_state_backend=filesystem \
  -e flink_taskmanager_numberOfTaskSlots=1 \
  -e flink_taskmanager_heap_mb=2048 \
  -e HOST=127.0.0.1 \
  -e PORT0=7001 \
  -e PORT1=7002 \
  -e PORT2=7003 \
  mesoshq/flink:1.1.2 taskmanager
```

### Via Mesos/Marathon
 
Start a standalone JobManager (you need to replace the `flink_recovery_zookeeper_quorum` variable with a valid setting for your cluster):

```
{
  "id": "/flink/jobmanager",
  "cmd": null,
  "cpus": 1,
  "mem": 1024,
  "disk": 0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "mesoshq/flink:1.1.2",
      "network": "HOST",
      "privileged": false,
      "parameters": [],
      "forcePullImage": true
    }
  },
  "args": ["jobmanager"],
  "env": {
    "flink_recovery_mode": "zookeeper",
    "flink_recovery_zookeeper_quorum": "172.17.10.101:2181",
    "flink_recovery_zookeeper_storageDir": "/data/zk"
  },
  "ports": [0, 0],
  "healthChecks": [
    {
      "portIndex": 0,
      "protocol": "TCP",
      "gracePeriodSeconds": 30,
      "intervalSeconds": 10,
      "timeoutSeconds": 3,
      "maxConsecutiveFailures": 1
    }
  ]
}
```

Start a TaskManager:

```
{
  "id": "/flink/taskmanager",
  "cmd": null,
  "cpus": 2,
  "mem": 4096,
  "disk": 0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "mesoshq/flink:1.1.2",
      "network": "HOST",
      "privileged": false,
      "parameters": [],
      "forcePullImage": true
    }
  },
  "args": ["taskmanager"],
  "env": {
    "flink_recovery_mode": "zookeeper",
    "flink_recovery_zookeeper_quorum": "172.17.10.101:2181",
    "flink_recovery_zookeeper_storageDir": "/data/zk",
    "flink_taskmanager_tmp_dirs": "/data/tasks",
    "flink_state_backend": "filesystem",
    "flink_blob_storage_directory": "/data/blobs",
    "flink_taskmanager_numberOfTaskSlots": "2",
    "flink_taskmanager_heap_mb": "4096"
  },
  "ports": [0, 0, 0],
  "healthChecks": [
    {
      "portIndex": 0,
      "protocol": "TCP",
      "gracePeriodSeconds": 30,
      "intervalSeconds": 10,
      "timeoutSeconds": 3,
      "maxConsecutiveFailures": 1
    }
  ]
}
```