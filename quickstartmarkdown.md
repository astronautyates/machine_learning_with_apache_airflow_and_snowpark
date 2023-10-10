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
  1. **Your project repository cloned to your computer.** For connection details about your Git repository, open the Repository and copy the `HTTPS` link provided near the top of the page. If you have at least one file in your repository then click on the green `Code` icon near the top of the page and copy the `HTTPS` link. Use that link in VS Code or your favorite IDE to clone the repo to your computer.
1. Docker
  1. **Docker Desktop on your laptop.**  We will be running Airflow as a container. Please install Docker Desktop on your desired OS by following the [Docker setup instructions](https://docs.docker.com/desktop/).

### What You’ll Build 
- A Machine Learning Pipeline called "Customer Analytics" that predicts customer lifetime value based on customer sentiment

<!-- ------------------------ -->
## Set up of environment
Duration: 2

First, Clone [this repository](https://github.com/astronomer/airflow-snowparkml-demo/tree/main) and navigate into its directory in terminal, before opening the folder up in the code editor of your choice! 

```
git clone https://github.com/astronautyates/SnowParkMLWorkshop
cd airflow-snowparkml-demo
```




