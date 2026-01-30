# GoodReads Data Pipeline

<img src="https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/goodreads.png" align="centre">

## Architecture 
![Pipeline Architecture](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/architecture.png)

The pipeline consists of several integrated modules:

 - GoodReads Python Wrapper
 - ETL Jobs
 - Redshift Warehouse Module
 - Analytics Module 

#### Overview
Data is captured in real time from the Goodreads API using a Python wrapper. The data collected from the API is stored on local disk and moved to the Landing Bucket on AWS S3. ETL jobs are written in Spark and scheduled in Airflow to run every 10 minutes.  

### ETL Flow

 - Data collected from the API is moved to landing zone S3 buckets.
 - The ETL job includes an S3 module which copies data from the landing zone to the working zone.
 - Once the data is moved to the working zone, a Spark job is triggered which reads the data and applies transformations. The dataset is repartitioned and moved to the Processed Zone.
 - The Warehouse module of the ETL jobs picks up data from the processed zone and stages it into Redshift staging tables.
 - Using the Redshift staging tables, an UPSERT operation is performed on the Data Warehouse tables to update the dataset.
 - ETL job execution is completed once the Data Warehouse is updated. 
 - An Airflow DAG runs data quality checks on all Warehouse tables once the ETL job execution is completed.
 - The Airflow DAG has analytics queries configured in a Custom Designed Operator. These queries are run and a subsequent Data Quality Check is performed on selected Analytics Tables.
 - DAG execution completes after these Data Quality checks.

## Environment Setup

### Hardware Used
EMR - This project utilizes a 3 node cluster with the following Instance Types:

    m5.xlarge
    4 vCore, 16 GiB memory, EBS only storage
    EBS Storage: 64 GiB

Redshift - For the Redshift component, a 2 Node cluster with Instance Types "dc2.large" is used.

### Setting Up Airflow

Detailed instructions on how to setup Airflow using AWS CloudFormation scripts are available in the documentation. This setup ensures a robust orchestration layer for the pipeline.

NOTE: This setup uses an EC2 instance and a Postgres RDS instance. Ensure you review the associated AWS charges before running the CloudFormation Stack. 

The project uses "sshtunnel" to submit Spark jobs using an SSH connection from the EC2 instance. This setup does not automatically install "sshtunnel" for Apache Airflow. It can be installed by running the following command: 

    pip install apache-airflow[sshtunnel]

Finally, copy the dag and plugin folders to the EC2 instance inside the Airflow home directory. Refer to the Airflow Connection documentation for setting up connections to EMR and Redshift from Airflow.

### Setting up EMR
Spinning up the EMR cluster follows standard AWS procedures. ETL jobs in this project use "psycopg2" to connect to the Redshift cluster for staging and warehouse queries. 

To install psycopg2 on EMR:

    sudo pip-3.6 install psycopg2

Since psycopg2 requires "postgresql-devel" and "postgresql-libs", you may need to install these dependencies first:

    sudo yum install postgresql-libs
    sudo yum install postgresql-devel

ETL jobs also use "boto3" to move files between S3 buckets. To install boto3, run:

    pip-3.6 install boto3 --user

Finally, as PySpark uses Python 2 as the default setup on EMR, set the environment variables to use Python 3:

    export PYSPARK_DRIVER_PYTHON=python3
    export PYSPARK_PYTHON=python3

Copy the ETL scripts to EMR to prepare the environment for job execution. 

### Setting up Redshift
You can follow the standard AWS guides to run a Redshift cluster or utilize the provided Infrastructure as Code (IaC) scripts to create the cluster automatically. 

## How to run 
Ensure the Airflow webserver and scheduler are running. 
Open the Airflow UI at http://[ec2-instance-ip]:[configured-port]

GoodReads Pipeline DAG
![Pipeline DAG](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/goodreads_dag.PNG)

DAG View:
![DAG View](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/DAG.PNG)

DAG Tree View:
![DAG Tree](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/DAG_tree_view.PNG)

DAG Gantt View: 
![DAG Gantt View](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/DAG_Gantt.PNG)

## Testing the Limits
The "goodreadsfaker" module in this project generates synthetic data used to test the ETL pipeline under heavy load.  

To test the pipeline, I used the faker module to generate 11.4 GB of data to be processed every 10 minutes. This includes ETL jobs, warehouse population, and analytical queries, which equates to approximately 68 GB/hour and 1.6 TB/day.

Source DataSet Count:
![Source Dataset Count](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/DatasetCount.PNG)

DAG Run Results:
![GoodReads DAG Run](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/DAG_tree_view.PNG)

Data Loaded to Warehouse:
![GoodReads Warehouse Count](https://github.com/san089/goodreads_etl_pipeline/blob/master/docs/images/WarehouseCount.PNG)

## Scenarios

- Data increase by 100x (Read vs Write):
    - Redshift: As an analytical database optimized for aggregation, it maintains high performance for read-heavy workloads.
    - EMR: The cluster size can be scaled horizontally to handle larger volumes of data processing.

- Pipelines running at 7am daily:
    - The DAG is currently scheduled for 10-minute intervals but can be reconfigured for a specific daily schedule. 
    - Data quality operators ensure integrity. Email triggers are configured to alert the team in case of failures.
    
- Availability for 100+ users:
    - Concurrency limits can be managed within the Amazon Redshift cluster. While the default is 50 parallel queries, additional clusters can be launched to meet business demand.

## Maintainer
This project is maintained by Vamsee Krishna Kotha.

Vamsee is a Data Scientist - AI with over 3 years of professional experience specializing in Python, SQL, R, and modern data engineering stacks.

- Email: vamseekrishna9201@gmail.com
- Key Skills: Python, SQL, R, Go, TypeScript, LangGraph, ADK
- Focus: Building scalable data pipelines and AI-driven analytical solutions.