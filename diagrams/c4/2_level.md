```mermaid
C4Container
  title Container Diagram for NeoBank Digital Platform

  Person(customer, "Bank Customer", "Mobile/Web user")
  Person_Ext(developer, "Third-party Developer", "Open Banking consumer")
  Person(employee, "Bank Employee", "Back-office staff")
  System_Ext(external, "External Services", "SMS, Email, KYC, Payment Networks")

  Enterprise_Boundary(platform, "NeoBank Digital Platform") {
      Boundary(cloud, "Cloud Environment (AWS)", "boundary") {

          Container(apiGateway, "API Gateway", "AWS API Gateway", "Entry point - OAuth2 authentication, rate limiting, routing")
          Container(monitoring, "Monitoring & Observability", "CloudWatch + Sentry", "Centralized logging, metrics, error tracking, and alerting")

          Container_Boundary(cloudServices, "Cloud Services") {
              Container(mobileBackend, "Mobile/Web Backend", "Node.js", "Customer-facing operations and business logic")
              Container(openBankingAPI, "Open Banking API", "Node.js", "PSD2/Open Banking compliant APIs")
              Container(fraudSync, "Fraud Detection (Sync)", "Node.js", "Real-time fraud validation blocking suspicious transactions")
              Container(fraudML, "Fraud Detection (ML)", "Python/TensorFlow", "Async ML-based pattern analysis and risk scoring")
          }

          Container_Boundary(cloudData, "Cloud Data Layer") {
              ContainerDb(cloudDB, "Cloud Database", "PostgreSQL", "Application data for cloud services")
              ContainerDb(dataWarehouse, "Data Warehouse", "AWS Redshift", "Historical analytics - 7 years")
              ContainerDb(dataLake, "Data Lake", "AWS S3", "Raw data for ML training and audit trail")
              SystemQueue(kafkaCloud, "Kafka Cluster", "AWS MSK", "Unified event streaming platform")
          }

          Container_Boundary(dataPipeline, "Data Pipeline Layer") {
              Container(etlService, "ETL Service", "AWS Glue", "Nightly batch: moves data from PostgreSQL and DB2 to Redshift and S3")
          }
      }

      Boundary(onprem, "On-Premise Environment", "boundary") {
          Container_Boundary(integration, "Integration & New Services") {
              Container(integrationLayer, "Legacy Integration Layer", "Node.js/ESB", "REST API wrapper exposing mainframe functions")
              Container(coreServices, "Core Banking Services", "Node.js", "New compliance-required services that must run on-prem")
              Container(cdcService, "CDC Service", "Debezium/IBM CDC", "Captures changes from DB2/ADABAS and publishes to Kafka")
          }

          Container_Boundary(legacy, "Legacy Banking System") {
              Container(mainframe, "Core Banking System", "COBOL on z/OS", "IBM iSeries mainframe - existing CBS")
              ContainerDb(db2, "DB2 Database", "DB2", "All transaction data and account balances")
              ContainerDb(adabas, "ADABAS Database", "ADABAS", "Customer information and profiles")
          }

          Container_Boundary(onpremData, "On-Prem Data Layer") {
              SystemQueue(kafkaOnprem, "Kafka Cluster", "Apache Kafka", "On-prem event streaming")
              ContainerDb(onpremDB, "On-Prem Database", "PostgreSQL", "Data for new on-prem services")
          }
      }
  }

  Rel(customer, apiGateway, "Uses banking services", "HTTPS/REST")
  Rel(developer, apiGateway, "Calls Open Banking APIs", "HTTPS/OAuth2")
  Rel(employee, apiGateway, "Manages operations", "HTTPS")

  Rel(apiGateway, mobileBackend, "Routes to", "HTTPS/REST")
  Rel(apiGateway, openBankingAPI, "Routes to", "HTTPS/REST")

  Rel(mobileBackend, cloudDB, "Reads/Writes", "SQL")
  %%Rel(mobileBackend, kafkaCloud, "Publishes events", "Kafka Producer")
  Rel(mobileBackend, integrationLayer, "Calls legacy", "REST")
  Rel(mobileBackend, external, "Notifications", "HTTPS")

  Rel(openBankingAPI, integrationLayer, "Fetches data", "REST")

  %%Rel(kafkaCloud, fraudML, "Consumes events", "Kafka Consumer")
  %%Rel(kafkaCloud, dataLake, "Streams to archive", "Kafka Connect S3")

  %%BiRel(kafkaCloud, kafkaOnprem, "Syncs events", "MirrorMaker 2")

  Rel(etlService, cloudDB, "Extracts", "JDBC")
  Rel(etlService, db2, "Extracts legacy data", "JDBC/CDC")
  Rel(etlService, dataWarehouse, "Loads transformed data", "Redshift COPY")
  Rel(etlService, dataLake, "Archives raw data", "S3 API")

  Rel(integrationLayer, mainframe, "Invokes", "CICS/MQ")
  Rel(integrationLayer, db2, "Queries", "JDBC")
  Rel(integrationLayer, adabas, "Queries", "ADABAS API")
  UpdateRelStyle(integrationLayer, mainframe, $textColor="gray", $lineColor="gray")
  UpdateRelStyle(integrationLayer, db2, $textColor="gray", $lineColor="gray")
  UpdateRelStyle(integrationLayer, adabas, $textColor="gray", $lineColor="gray")

  Rel(coreServices, onpremDB, "Reads/Writes", "SQL")
  UpdateRelStyle(coreServices, onpremDB, $textColor="gray", $lineColor="gray")
  %%Rel(coreServices, kafkaOnprem, "Publishes events", "Kafka Producer")

  Rel(db2, cdcService, "Transaction log", "CDC")
  Rel(adabas, cdcService, "Change log", "CDC")
  %%Rel(cdcService, kafkaOnprem, "Publishes changes", "Kafka Producer")

  Rel(mainframe, db2, "Transactions", "DB2")
  Rel(mainframe, adabas, "Customer data", "ADABAS")

  UpdateElementStyle(apiGateway, $fontColor="white", $bgColor="#1168bd", $borderColor="#0b4884")
  UpdateElementStyle(integrationLayer, $fontColor="white", $bgColor="#e8900e", $borderColor="#b5700b")
  UpdateElementStyle(mainframe, $fontColor="white", $bgColor="#666", $borderColor="#333")
  UpdateElementStyle(kafkaCloud, $fontColor="white", $bgColor="#8B0000", $borderColor="#5C0000")
  UpdateElementStyle(kafkaOnprem, $fontColor="white", $bgColor="#8B0000", $borderColor="#5C0000")

  UpdateRelStyle(customer, apiGateway, $textColor="#92BDA3", $lineColor="#92BDA3")
  UpdateRelStyle(apiGateway, mobileBackend, $textColor="#92BDA3", $lineColor="#92BDA3")



  UpdateLayoutConfig($c4ShapeInRow="5", $c4BoundaryInRow="3")
  ```