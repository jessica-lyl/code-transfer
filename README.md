# Leak detection

Create an API to train a model of classification and store training data as well as performances of the model.

## 🔍️ Table of content

- [Documentation](#📄-documentation)
- [Requirements](#💻-requirements)
- [Installation](#⚙️-installation)
- [Usage](#📈-usage)
- [MLFlow/Google Cloud](#🗃️-mlflowgoogle-cloud)
- [CI/CD](#🧱-cicd)
- [Deployment](#🚀-deployment)
- [How to contribute ?](#🛠️-how-to-contribute)


### FastAPI interface
![GIF 1](gifs/tool-interface.png)


## 📄 Documentation

**Documentation**:
<br> [https://ai.veolia.tech/guidelines/operationalization/streamlit/streamlit/](https://ai.veolia.tech/guidelines/operationalization/streamlit/streamlit/)
<br> [https://ai.veolia.tech/guidelines/operationalization/lab-iris/intro/](https://ai.veolia.tech/guidelines/operationalization/lab-iris/intro/)
<br> [API documentation](https://pudong-leak-detection-veolia-com-asia-special-pr-76e8b218fd571a.gitlab.io/)


**Source code**: [https://gitlab.com/veolia.com/asia/special-projects-asia/sandbox-data/pudong-leak-detection](https://gitlab.com/veolia.com/asia/special-projects-asia/sandbox-data/pudong-leak-detection)



## 💻 Requirements

### Python version
* Main supported version : <strong>3.11</strong> <br>

Please make sure you have one of these versions installed to be able to run the app on your machine.


## ⚙️ Installation


### Create a virtual environment (optional)
We strongly advise to create and activate a new virtual environment, to avoid any dependency issue.

For example with conda:
```bash
pip install conda; conda create -n streamlit_leak_detection python=3.11; conda activate streamlit_leak_detection
```

Or with virtualenv:
```bash
pip install virtualenv; python3.11 -m virtualenv streamlit_leak_detection --python=python3.7; source streamlit_leak_detection/bin/activate
```

### Clone the repository
Clone the repository locally:
```bash
git clone git@gitlab.com:veolia.com/asia/special-projects-asia/sandbox-data/pudong-leak-detection.git
```

### Install package
Install the package from requirements.txt:
```bash
pip install -r app/requirements.txt
```


## 📈 Usage

Before you can use your code, please check the following conditions:

* <strong>Config</strong>:
<br>[app\src\config.py](app\src\config.py) : indicate the different informations about your BigQuery table where is stored your input data.

![Picture 1](gifs/config.png)
<br>This dataset should contain **input columns**, **target column** with the name **'leaks'** and must be in the wide format like on the following example :

<img src="gifs/BQ_table.png" width="600">

* <strong>Inputs</strong>:
<br>[app\src\config.py](app\src\config.py) : indicate the different features or inputs of your model in this file

![Picture 2](gifs/features.png)

<br>[app/src/datamodels.py](app/src/datamodels.py) : inthe class 'PipesFeatures', indicate all the features with their type that you want to use

![Picture 3](gifs/datamodels.png)

* <strong>Cloud Storage</strong>:
<br>Create a bucket with the following name : GCP_PROJECT_ID-DEFAULT_ARTIFACTS_BUCKET_NAME_GENERIC, where DEFAULT_ARTIFACTS_BUCKET_NAME_GENERIC is defined in the file [app/src/artifacts_management.py](app/src/artifacts_management.py)

<img src="gifs/artifacts.png" width="600">

<br>The bucket must have this form, and the location is defined in the file [app/src/utils/data_io_services/cloud_storage.py](app/src/utils/data_io_services/cloud_storage.py) :

<img src="gifs/bucket.png" width="600">

* <strong>Firestore</strong>:
<br>Create a database in Firestore with the name DEFAULT_METADATA_DATABASE_NAME_GENERIC defined in the file [app/src/artifacts_management.py](app/src/artifacts_management.py)

<img src="gifs/firestore.png" width="600">

For the last 3 elements, you must give the following access to the service account with which you deploy the application:
- BigQuery : BigQuery Data Viewer on the table you are storing your data
- Cloud Storage : Give access to write in the bucket
- Firestore : Grant access for writing in the database : you can use the following command in Cloud Shell Terminal adapting the PROJECT_ID, DATABASE_ID and the EMAIL (TITLE and DESCRIPTION are facultative) :

```bash
PROJECT_ID=cn-ops-asia-cloudscada
EMAIL='bigquery@cn-ops-asia-cloudscada.iam.gserviceaccount.com'
DATABASE_ID=plastic-experiment
TITLE=plastic-experiment-metadata
DESCRIPTION=database_for_metadata
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member="serviceAccount:$EMAIL" \
--role="roles/datastore.user" \
--condition='expression=resource.name=="projects/cn-ops-asia-cloudscada/databases/plastic-experiment",title=plastic-experiment-metadata,description=database_for_metadata'
```

* <strong>MLFlow</strong>:
<br>Change the parameters related to MLFlow (mlflow_tracking_uri and mlflow_tracking_token) in the file [app/src/config.py](app/src/config.py) :

<img src="gifs/mlflow.png" width="600">

mlflow_tracking_token must be created via GitLab in Settings --> Access Tokens and Add new token with Scopes "API"

Once realized, run the following command to open the app in your default web browser:


Now you can use the optimization tool.


* <strong>Pytest</strong>:
<br>Create the file [app/data/dataset_test.parquet](app/data/dataset_test.parquet) used for the tests

```python
from app.src.config import (DATASET_ID, PROJECT_ID, TABLE_ID, COLUMNS)
sql = f"""
    SELECT {', '.join(COLUMNS)}, `leaks`
    FROM `{PROJECT_ID}.{DATASET_ID}.{TABLE_ID}`
    LIMIT 100000
"""

df = pd.read_gbq(sql, project_id=PROJECT_ID)

df.to_parquet('../app/data/dataset_test.parquet')
```

<br>Adapt your examples in the file [app/tests/default_test_cases.py](app/tests/default_test_cases.py)

<img src="gifs/defaults_cases.png" width="600">

<br>Use the following command to trigger your tests
```bash
python3 -m pytest app/
```

* <strong>Interactive docs</strong>:
To visualize interactive openapi specification, launch locally your api server :
```bash
uvicorn app.main:app --reload
```

Then open your browser and go to
```
http://127.0.0.1:8000/docs
```


## 🗃️ MLFlow/Google Cloud

MLFlow on GitLab is a powerful tool designed to facilitate the management of machine learning projects and their associated models and metrics. By leveraging MLFlow, we can conveniently store all our models and their corresponding metrics, enabling we to effectively track the history and evolution of our models over time. This capability empowers us to compare different versions of our models, monitor performance improvements, and make well-informed decisions based on historical data. With MLFlow on GitLab, we can ensure reproducibility, foster collaboration among team members, and efficiently manage our machine learning projects. Additionally, MLFlow allows we to organize models based on the target selected by the user, providing a flexible and customizable approach to model storage and retrieval. By utilizing MLFlow on GitLab, we can streamline our machine learning workflow, enhance project transparency, and drive impactful results.

On Google Cloud, we utilize a similar process to store the datasets used for training, as well as the history of the recommendations. By leveraging the capabilities of Google Cloud, we can efficiently manage and store the datasets, ensuring their availability and accessibility for training purposes. Additionally, by keeping a record of the recommendation history, we can track the performance and effectiveness of the recommendations over time. This allows for continuous improvement and optimization of the recommendation system. With Google Cloud, we can seamlessly integrate data storage and management into my machine learning workflow, enabling efficient collaboration and reproducibility of results.


## 🧱 CI/CD

CI/CD on GitLab refers to the continuous integration and continuous deployment process implemented on the GitLab platform. It involves automating the build, testing, and deployment of software applications. With GitLab CI/CD, developers can push their code changes to a Git repository, triggering a pipeline that automatically builds, tests, and deploys the application. This process ensures that code changes are thoroughly tested and deployed to production environments in a consistent and efficient manner. GitLab CI/CD also provides features like parallel testing, environment management, and integration with various tools and services, making it a comprehensive solution for implementing a robust CI/CD workflow.

We also use the file .pre-commit-config.yml in order to impose rules on commits. This makes it possible to ensure that the CI/CD pipelines will go through well during the push. You can install the pre-commit by performing these two commands:
```bash
pip install pre-commit
```
```bash
pre-commit install
```

## 🚀 Deployment

To deploy on Cloud Run, you need to modify the information contained in the following file to adapt it to your project: `.gitlab-ci.yml`, especially the following fields:

`GCP_PROJECT_NAME_GENERIC`

`GCP_DEFAULT_LOCATION`

`GCP_ARTIFACT_REGISTRY_REPOSITORY`

`GCP_VPC_NETWORK_NAME`

`GCP_CLOUD_RUN_SERVICE_NAME`

`GCP_CLOUD_RUN_MEMORY`

`GCP_CLOUD_RUN_MIN_INSTANCES`

`GCP_CLOUD_RUN_TIMEOUT`


## 🛠️ How to contribute ?

Here is the important information about the repository:

-**app** :

This folder contains all the code necessary for the web app to function on Streamlit, as well as the tests.

-**.gitlab-ci.yml** :

The `.gitlab-ci.yml` file defines the structure and order of the pipelines and determines:

- What to execute using [GitLab Runner](https://docs.gitlab.com/runner/).
- What decisions to make when specific conditions are encountered. For example, when a process succeeds or fails.


-**.pre-commit-config.yaml** :

The .pre-commit-config.yaml file is a configuration file used by the pre-commit tool to define pre-commit hooks to run before each commit in a project. It specifies the commands to run, the files to check, and the actions to take if any issues are found.

-**app.yaml** :

You configure your App Engine app's settings in the `app.yaml` configuration file. Your config file must specify at least a runtime entry.

Each service in your app has its own `app.yaml` file, which acts as a descriptor for its deployment.

-**cloudbuild.yaml** :

Cloud Build is a service that executes your builds on Google Cloud.

Cloud Build can import source code from a variety of repositories or cloud storage spaces, execute a build to your specifications, and produce artifacts such as Docker containers or Java archives.

-**Dockerfile** :

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. This page describes the commands you can use in a Dockerfile.

