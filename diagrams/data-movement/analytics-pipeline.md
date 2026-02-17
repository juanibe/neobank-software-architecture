```mermaid
sequenceDiagram
    Customer           ->>   Mobile Backend   :         Transfer
    Mobile Backend     -->>  RabbitMQ         :         Publish "transfer.success" 
    Mobile Backend     ->>   Customer         :         Success
    RabbitMQ           -->>  Kinesis          :         Stream Events
    Kinesis            -->>  Stream Processor :         Process one by one
    Stream Processor   -->>  Data Lake (S3)   :         Write Parquet
```