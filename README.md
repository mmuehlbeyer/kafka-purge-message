# kafka-purge-message

short overview how to delete messages from within a kafka topic

prerequisits

* a running kafka cluster --> spin up a local cluster with the docker-compose.yml in the repo
* kafka client libraries installed

### OPTIONAL: start the kafka cluster (kraft based)

```bash
export CP_VERSION=7.5.3
docker-compose up -d
```

### create a topic

```bash
kafka-topics --create --topic demo-purge --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1
```

### produce some messages

```bash
seq -f "demo-msg-del_%g ${RANDOM}" 20 | kafka-console-producer --broker-list localhost:9092 --topic demo-purge
```


### check the data with kcat


```bash
kcat -b localhost:9092 -t demo-purge -C -f '\nValue (%S bytes): %s %T\tPartition: %p\tOffset: %o\n--'

Value (20 bytes): demo-msg-del_1 28547 1707389871693	Partition: 0	Offset: 0
--
Value (20 bytes): demo-msg-del_2 28547 1707389871700	Partition: 0	Offset: 1
--
Value (20 bytes): demo-msg-del_3 28547 1707389871700	Partition: 0	Offset: 2
--
Value (20 bytes): demo-msg-del_4 28547 1707389871700	Partition: 0	Offset: 3
--
Value (20 bytes): demo-msg-del_5 28547 1707389871700	Partition: 0	Offset: 4
--
Value (20 bytes): demo-msg-del_6 28547 1707389871700	Partition: 0	Offset: 5
--
Value (20 bytes): demo-msg-del_7 28547 1707389871700	Partition: 0	Offset: 6
--
Value (20 bytes): demo-msg-del_8 28547 1707389871700	Partition: 0	Offset: 7
--
Value (20 bytes): demo-msg-del_9 28547 1707389871700	Partition: 0	Offset: 8
--
Value (21 bytes): demo-msg-del_10 28547 1707389871700	Partition: 0	Offset: 9
--
Value (21 bytes): demo-msg-del_11 28547 1707389871700	Partition: 0	Offset: 10
--
Value (21 bytes): demo-msg-del_12 28547 1707389871700	Partition: 0	Offset: 11
--
Value (21 bytes): demo-msg-del_13 28547 1707389871700	Partition: 0	Offset: 12
--
Value (21 bytes): demo-msg-del_14 28547 1707389871700	Partition: 0	Offset: 13
--
Value (21 bytes): demo-msg-del_15 28547 1707389871700	Partition: 0	Offset: 14
--
Value (21 bytes): demo-msg-del_16 28547 1707389871701	Partition: 0	Offset: 15
--
Value (21 bytes): demo-msg-del_17 28547 1707389871701	Partition: 0	Offset: 16
--
Value (21 bytes): demo-msg-del_18 28547 1707389871701	Partition: 0	Offset: 17
--
Value (21 bytes): demo-msg-del_19 28547 1707389871701	Partition: 0	Offset: 18
--
Value (21 bytes): demo-msg-del_20 28547 1707389871701	Partition: 0	Offset: 19
```

offset from 0 -19

### prep the deletion

create an offset file offset-del.json with the following


```json
{"partitions":[{"topic": "demo-purge", "partition": 0,"offset": 5}],"version":1}
```

will delete everything up to partition offset 5

### delete the records

```bash
kafka-delete-records --bootstrap-server localhost:9092 --offset-json-file offset-del.json
Executing records delete operation
Records delete operation completed:
partition: demo-purge-0	low_watermark: 5
```

### check with kcat

´´´bash
kcat -b localhost:9092 -t demo-purge -C -f '\nValue (%S bytes): %s %T\tPartition: %p\tOffset: %o\n--'

Value (20 bytes): demo-msg-del_6 28547 1707389871700	Partition: 0	Offset: 5
--
Value (20 bytes): demo-msg-del_7 28547 1707389871700	Partition: 0	Offset: 6
--
Value (20 bytes): demo-msg-del_8 28547 1707389871700	Partition: 0	Offset: 7
--
Value (20 bytes): demo-msg-del_9 28547 1707389871700	Partition: 0	Offset: 8
--
Value (21 bytes): demo-msg-del_10 28547 1707389871700	Partition: 0	Offset: 9
--
Value (21 bytes): demo-msg-del_11 28547 1707389871700	Partition: 0	Offset: 10
--
Value (21 bytes): demo-msg-del_12 28547 1707389871700	Partition: 0	Offset: 11
--
Value (21 bytes): demo-msg-del_13 28547 1707389871700	Partition: 0	Offset: 12
--
Value (21 bytes): demo-msg-del_14 28547 1707389871700	Partition: 0	Offset: 13
--
Value (21 bytes): demo-msg-del_15 28547 1707389871700	Partition: 0	Offset: 14
--
Value (21 bytes): demo-msg-del_16 28547 1707389871701	Partition: 0	Offset: 15
--
Value (21 bytes): demo-msg-del_17 28547 1707389871701	Partition: 0	Offset: 16
--
Value (21 bytes): demo-msg-del_18 28547 1707389871701	Partition: 0	Offset: 17
--
Value (21 bytes): demo-msg-del_19 28547 1707389871701	Partition: 0	Offset: 18
--
Value (21 bytes): demo-msg-del_20 28547 1707389871701	Partition: 0	Offset: 19
% Reached end of topic demo-purge [0] at offset 20
