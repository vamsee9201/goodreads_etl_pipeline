# Data Warehouse Infrastructure

This repository contains the configuration and scripts required to manage the data warehouse environment.

## warehouse_config.cfg file in warehouse folder contains  

    [AWS]  
    KEY=<AWS-KEY>
    SECRET=<AWS-SECRET-KEY>
      
    [CLUSTER]  
    HOST='<Redshift Cluster Endpoint>'  
    DB_NAME='<db-name>'  
    DB_USER='<db-user-name>'  
    DB_PASSWORD='<db-password>'  
    DB_PORT=<db-port, default 5439>  
      
    [IAM_ROLE]  
    ARN=<Redshift ARN role>
      
      
    [STAGING]  
    SCHEMA=<Warehouse-staging-schema>  
      
    [WAREHOUSE]  
    SCHEMA=<Warehouse-schema>  
      
      
    [BUCKET]  
    LANDING_ZONE=<landing-zone-bucket>  
    WORKING_ZONE=<working-zone-bucket>
    PROCESSED_ZONE=<processed-zone-bucket>

## Maintainer

Vamsee Krishna Kotha
Data Scientist - AI
Email: vamseekrishna9201@gmail.com

## About the Developer

Vamsee Krishna Kotha is a Data Scientist - AI with over 3 years of professional experience in the industry. With a strong background in Python, SQL, R, Go, and TypeScript, he specializes in building robust data architectures and AI-driven solutions. His expertise includes working with LangGraph and ADK to streamline complex data workflows and warehouse management.