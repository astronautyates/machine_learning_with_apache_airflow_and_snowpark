author: Michael Gregory, George Yates
id: ml_with_snowpark_and_apache_airflow
summary: This is a sample Snowflake Guide
categories: data-engineering,architecture-patterns,partner-integrations
environments: web
status: Development 
feedback link: https://github.com/Snowflake-Labs/sfguides/issues
tags: Data Engineering, Snowpark, Airflow, Machine Learning, AI


# Machine Learning with Snowpark and Apache Airflow
<!-- ------------------------ -->
## Overview 
Duration: 30

Snowpark ML (in public preview) is a python framework for Machine Learning workloads with Snowpark. Currently Snowpark ML provides a model registry (storing ML tracking data and models in Snowflake tables and stages), feature engineering primitives similar to scikit-learn (ie. LabelEncoder, OneHotEncoder, etc.) and support for training and deploying certain model types as well as deployments as user-defined functions (UDFs).

This virtual hands-on lab demonstrates how to use Apache Airflow to orchestrate a machine learning pipeline leveraging Snowpark ML for feature engineering and model tracking. While Snowpark ML has its own support for models similar to scikit-learn this code demonstrates a "bring-your-own" model approach showing the use of open-source scikit-learn along with Snowpark ML model registry and model serving in an Airflow task rather than Snowpark UDF. It also shows the use of the Snowflake XCOM backend which supports security and governance by serializing all task in/output to Snowflake tables and stages while storing in the Airflow XCOM table a URI pointer to the data. 

This workflow includes:

- Sourcing structured, unstructured and semistructured data from different systems
- Extracting, transforming and loading with the Snowpark Python provider for Airflow
- Ingesting with Astronomer's python SDK for Airflow
- Audio file transcription with OpenAI Whisper
- Natural language embeddings with OpenAI Embeddings and the Weaviate provider for Airflow
- Vector search with Weaviate
- Sentiment classification with LightGBM
- ML model management with Snowflake ML

Let’s get started. 

### Prerequisites
This guide assumes you have a basic working knowledge of Python, Airflow, and Snowflake

### What You’ll Learn 
- How to use Airflow to manage your Machine Learning Operations
- How to leverage Snowpark's compute for your Machine Learning workflows
- How to use Snowpark & Airflow together to create horizontally and vertically scalable ML pipelines

### What You’ll Need 
You will need the following things before beginning:

1. Snowflake
  1. **A Snowflake Account.**
  1. **A Snowflake User created with appropriate permissions.** This user will need sys/accountadmin level permissions to create and manipulate the necessary databases.
  2. Follow [these instructions](https://docs.snowflake.com/en/developer-guide/udf/python/udf-python-packages#using-third-party-packages-from-anaconda) to enable Anaconda Packages in your trial account
1. GitHub
  1. **A GitHub Account.** If you don’t already have a GitHub account you can create one for free. Visit the [Join GitHub](https://github.com/join) page to get started.
  1. **Download the Project's GitHub Repository.** To do this workshop, you'll need to download the following Repo onto your local machine: https://github.com/astronomer/airflow-snowparkml-demo/tree/main
1. Integrated Development Environment (IDE)
  1. **Your favorite IDE with Git integration.** If you don’t already have a favorite IDE that integrates with Git I would recommend the great, free, open-source [Visual Studio Code](https://code.visualstudio.com/).
1. Docker
  1. **Docker Desktop on your laptop.**  We will be running Airflow as a container. Please install Docker Desktop on your desired OS by following the [Docker setup instructions](https://docs.docker.com/desktop/).

### What You’ll Build 
- A Machine Learning Pipeline called "Customer Analytics" that predicts customer lifetime value based on customer sentiment

<!-- ------------------------ -->
## Set up of environment and Repo Overview
Duration: 2

First, Clone [this repository](https://github.com/astronomer/airflow-snowparkml-demo/tree/main) and navigate into its directory in terminal, before opening the folder up in the code editor of your choice! 

```
git clone https://github.com/astronautyates/SnowParkMLWorkshop
cd airflow-snowparkml-demo
```

Since we're using so many different tools installed via the requirements/packages files, it's worth going through them so you understand the systems being used in the 'Customer Analytics' DAG

Repository Contents Overview:

Requirements.txt
/tmp/airflow_provider_weaviate-1.0.0-py3-none-any.whl: This beta package allows us to connect to Weaviate for embedding generation. 

/tmp/astro_provider_snowflake-0.0.0-py3-none-any.whl: This beta package allows us to connect to Snowpark.  

snowflake-ml-python==1.0.7: A package that provides machine learning functionalities or integrations for Snowflake within Python.

pandas~=1.5: A powerful and flexible open-source data analysis and manipulation library for Python.

scikit-learn==1.3.0: A machine learning library in Python that provides simple and efficient tools for data analysis and modeling.

We will also show how to use a Python virtual environment to simplify Snowpark requirements if that is necessary in your Airflow environment.  For that we will create a requirements-snowpark.txt file with the following: 

psycopg2-binary: A standalone package that provides a PostgreSQL adapter.

snowflake_snowpark_python[pandas]>=1.5.1: An extension of the Snowflake Python Connector that provides an intuitive, pythonic API for querying and processing data in Snowflake. 

/tmp/astro_provider_snowflake-0.0.0-py3-none-any.whl: This provider simplifies the process of making secure connections to Snowflake, the process of building Airflow tasks with Snowpark code and the passing of Snowpark dataframes between tasks. 

virtualenv: A tool in Python used to create isolated Python virtual environments, which we’ll need to create a 3.8 Python virtual environment to connect to Snowpark


Packages.txt

Build-essential: A collection of essential development tools, including the GNU Compiler Collection (GCC), the GNU Debugger (GDB), and other libraries and tools. It is required to compile and install many other software packages, including FFmpeg.

ffmpeg: A free and open-source software project consisting of a suite of libraries and programs for handling video, audio, and other multimedia files and streams. We’ll use this to transcribe support calls with OpenAI Whisper.

Step 2: 

Navigate to the .env file and update the AIRFLOW_CONN_SNOWFLAKE_DEFAULT with your own credentials. These will be used to connect to Snowflake. The Snowflake account field of the connection should use the new ORG_NAME-ACCOUNT_NAME format as per Snowflake Account Identifier policies. The ORG and ACCOUNT names can be found in the confirmation email or in the Snowflake login link (ie. https://xxxxxxx-yyy11111.snowflakecomputing.com/console/login) Do not specify a region when using this format for accounts. If you need help finding your ORG_NAME-ACCOUNT_NAME, use [this guide](https://docs.snowflake.com/en/user-guide/admin-account-identifier) to find them. 

NOTE: Database and Schema names should remain as DEMO, and the warehouse name should remain COMPUTE_WH. This is because the Customer_Analytics DAG will create tables with these names and reference them throughout the DAG, so if you use different names you'll need to alter the DAG code to reflect them. 

AIRFLOW_CONN_SNOWFLAKE_DEFAULT='{"conn_type": "snowflake", "login": "<USER_NAME>", "password": "<PASSWORD>", "schema": "DEMO", "extra": {"account": "<ORG_NAME>-<ACCOUNT_NAME>", "warehouse": "COMPUTE_WH", "database": "DEMO", "region": "", "role": "ACCOUNTADMIN", "authenticator": "snowflake", "session_parameters": null, "application": "AIRFLOW"}}'

NOTE: The use of ACCOUNTADMIN in this demo is only to simplify setup of the quickstart. It is, of course, not advisable to use this role for production.

<!-- ------------------------ -->
## Start Environment and Run DAG

Now that you've gotten your environment set up, open up a terminal window in your 'airflow-snowparkml-demo' folder and run the following command: 
