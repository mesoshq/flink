# A Docker image for Apache Flink (1.0.3)

A base image for creating Apache Flink clusters. Usable to create jobmanagers or taskmanagers.

## Usage

### Configuration properties

You can set every [https://ci.apache.org/projects/flink/flink-docs-release-1.0/setup/config.html](config property) by passing it as lowercase environment variable in the Marathon app definition's `env` map, where the variable name is the original property with `_` instead of `.`, prepended by `flink_`.

For example, if you'd want to set `recovery.mode`, you'd have to specify `flink_recovery_mode` as environment variable (e.g. `-e flink_recovery_mode=zookeeper`) with an appropriate value.

Those environment variables will be automatically added to the `flink-config.yaml` file by the `docker-entrypoint.sh` script before the startup.

## Examples

Start a JobManager with ZooKeeper storage:

```
docker run -d 
       -e flink_recovery_mode=zookeeper \
       -e flink_recovery_zookeeper_quorum=172.17.10.101:2181 \
       -e flink_recovery_zookeeper_storageDir=/data/zk \
       --name JobManager \
       --net=host \ 
       mesoshq/flink:0.1.0 jobmanager
```

Start a TaskManager:

```
docker run -d 
       -e flink_recovery_mode=zookeeper \
       -e flink_recovery_zookeeper_quorum=172.17.10.101:2181 \
       -e flink_recovery_zookeeper_storageDir=/data/zk \
       -e flink_taskmanager_tmp_dirs=/data/tasks \
       -e flink_blob_storage_directory=/data/blobs \
       -e flink_state_backend=filesystem \
       -e flink_taskmanager_numberOfTaskSlots=1 \
       -e flink_taskmanager_heap_mb=2048 \
       --name JobManager \
       --net=host \ 
       mesoshq/flink:0.1.0 jobmanager
```
