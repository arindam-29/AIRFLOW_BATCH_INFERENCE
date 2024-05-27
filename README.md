# AIRFLOW_BATCH_INFERENCE
Build a system with Airflow for Scheduled Batch Inference Pipelines

Lets build an AI-based news aggregation and allows users to subscribe to various topic feeds that we will populate with news articles from various news publication RSS feeds. As we are just starting out, we only have a few Finance-related topic feeds ("Stock", "Crypto", "IPO", etc.) and only one news source so far (CNBC). We need to build an airflow DAG that will get new articles each day from the CNBC Finance RSS feed, categorize these articles into subtopics, and save those categories so our feed backend can serve article links to each user's feed.

To do this we will create an Airflow DAG with three tasks:
1.	Data Loader: this task retrieves news articles from the CNBC RSS feed, which is the raw data we will further process in the next stages.
2.	Text Classification: this task loads a pre-trained NLP model (pytorch) for zero-shot classification. The model predicts categories based on a hard-coded set of labels that we have pre-defined in `ml_pipeline/model_prediction/model_predict.py`
3.	Results Aggregation: This task takes the array of individual predictions and groups the articles by their predicted topic

## Code Overview:

First take a look at the dags/news_classifier.py file. There are three tasks in this DAG, each task will be executed sequentially. Tasks 1 and 2 use DockerOperator which means the tasks will be executed in a Docker Container. Task 3 uses PythonOperator which will run a python script.

![alt text](/screenshots/image.png)

### Lets look at each task.

Task 1 is in ml_pipline/data_loader/data_load.py. You can see it downloads the news feed data from the CNBC rss feed (rss feed actually serves an XML file) and also parses the file. After some simple pandas processing, it saves the raw data in a CSV file. Remember this is running in a Docker container, which has its own filesystem, but because we set up the mount previously, the data will also be written to your local filesystem. More on that later.

Task 2 is in ml_pipline/model_prediction/model_predict.py. You can see it loads a model ("valhalla/distilbart-mnli-12-1") as well as the CSV from task 1. In a production pipeline, the data might instead be loaded from cloud storage like S3, but the data source is abstracted in either case and isn't materially different. After some basic preprocessing the prediction is made on the input data and saved to an output json file.

Task 3 is actually a python function and it lives in dags/utils.py. It's a simple task that does some post processing. It loads the json file from Task 2 and aggregates the list of predictions so that they are grouped by their predicted label.

## Docker Desktop:

Ensure that the Docker (ie. background process) is running and that your Docker Engine has sufficient memory. Airflow runs as several services which will each need to take some memory resources from your host machine. I was able to run the demo with 4 GB of memory. To change your Docker memory resources, open Docker Desktop, hit the settings in the upper right corner. From the settings screen, select Resources and increase the memory resources for Docker if needed (recommend at least 4GB):

![alt text](/screenshots/image1.png)

## Add Project source path:

Open dags/news_classifier.py file and replace the placeholder with the path to the demo directory in your local filesystem. This directory will be mounted to your Docker container, which binds the docker container filesystem to the local filesystem so that any files written by the docker container will show up on your local machine (and VSCode). 

If you don't know the path, right click in the file tree on the left and "Copy Path" making sure no folders are highlighted so you get the root directory

![alt text](/screenshots/image2.png)

## Prepare to run AirFlow:

### get docker-compose.yaml:

For details review: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.1/docker-compose.yaml'
```
```bash
echo -e "AIRFLOW_UID=50000" > .env
```
```bash
docker compose up airflow-init
```
```bash
docker compose up
```

### Airflow web UI:

Open the Airflow web UI in your browser at http://localhost:8080. Sign in with username:airflow password:airflow

Search for the DAG: financial_news

Trigger the DAG. The DAG will take a few minutes to run (especially the second task which does inference). You can monitor your task progress by viewing the graph or logs for each task

![alt text](/screenshots/image4.png)

You can also find the logs locally since we mounted our filesystem to the worker docker containers

When all tasks have successfully completed, you should see the data saved in the data folder.

![alt text](/screenshots/image5.png)

## Tips:

"Format Document" on the json file to make it more readable

When you are finished working and want to clean up your environment, run
```bash
docker compose down --volumes --rmi all
```
