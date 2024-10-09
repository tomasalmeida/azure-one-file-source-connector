# Proposal

Create a new Blob Storage in Azure with one file, use it as a source of data for a kafka topic and updating the file, see the new data goes to the topic

## Start

```shell
    ./start.sh
```

## The resources

Follow https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal to create the container. I need to download the storage explorer to navigate and update files. As per our config, the files should be inside a folder called topics. Add the v1 file to that folder

## Check the logs

```shell
    docker compose logs connect -f
```

## Create the connector

```shell
curl -X PUT -H "Accept:application/json" \
  -H  "Content-Type:application/json" http://localhost:8083/connectors/blob-storage-source/config \
  -d '{
      "connector.class"          :           "io.confluent.connect.azure.blob.storage.AzureBlobStorageSourceConnector",
      "tasks.max"                :           "1",
      "confluent.topic.replication.factor" : "1",
      "confluent.topic.bootstrap.servers":   "kafka:9092",
      
      "mode"               : "GENERIC",
      "topic.regex.list": "topic-unique-file:topics/file.json",
      "partitioner.class"  : "io.confluent.connect.storage.partitioner.DefaultPartitioner",
      "format.class"       : "io.confluent.connect.cloud.storage.source.format.CloudStorageJsonFormat",
      "topics.dir"         : "topics",


      "azblob.account.name"     : "storage-account-name-CHANGE-IT",
      "azblob.account.key"      : "storage-account-key-CHANGE-IT",
      "azblob.container.name"   : "storage-container-name-CHANGE-IT"
      }' | jq
```

## Verify the connect offset

```shell
    kafka-console-consumer --topic connect-offsets --bootstrap-server localhost:19092 --property print.key=true --from-beginning 
```

## Update the file

Drop the new version of the file to the topics folder. Wait and nothing will happen :-) 

# Conclusions

As listed in the [documentation](https://docs.confluent.io/kafka-connectors/azure-blob-storage-source/current/generalized/overview.html#limitations):

> The connector won’t reload data during the following scenarios:
> * Renaming a file which the connector has already read.
> * Uploading a newer version of a file with a new record.

So uploading a new version of an already read file does nothing.

## Shutdown

```shell
    docker compose down -v
```

## References and inspirations

Thanks to 
* https://github.com/rjmfernandes/kafkaAzureBackupRestore - with the basics to run the cluster and set the Azure connector
* https://github.com/vdesabou/kafka-docker-playground/tree/master/connect/connect-azure-blob-storage-source - always a good inspiration
* https://docs.confluent.io/kafka-connectors/azure-blob-storage-source/current/backup-and-restore/overview.html
* 