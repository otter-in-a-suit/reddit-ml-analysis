# Reddit Analysis
A collection of Python scripts that read the top posts from popular subreddits and stores them as JSON. JSONs will be uploaded to Google Cloud storage.

A Apache Beam / Google Data Flow pipeline reads the data from GCS, applies the Cloud Vision API for image label detection and writes the results back to BigQuery.

A separate Beam pipeline, written in Java, can be used to stream messages directly to BigQuery from PubSUb without parsing the data using ML. 

A Python script that downloads images from a certain subreddit, stores them on GCS, and prepares a CSV for Cloud AutoML.

A Python script that sends an image to AutoML for classification.

Most programs require a Service Account JSON key.

This is a work-in progress project for my [blog](https://otter-in-a-suit.com/blog).

### Prerequisites
* Python 2.7.3
* A Google Cloud project
* An existing GCS bucket
* An existing AutoML model for automl.automl_rest.py

## Install
Install dependencies:

```
pip install --upgrade google-cloud-storage
pip install --upgrade praw
pip install --upgrade google-cloud-vision
```

## Configuration

```
cp config_example.py config.py
vim config.py
```

Set your reddit API and GCP keys here. Create the GCS bucket before via the Cloud Console.

If you plan to use PubSub, enable the corresponding option.

## PubSub
If PubSub is enabled, the script will write the message to PubSub as well. Create a topic as follows -
```
TOPIC=your_topic_name
SUB=your_sub_name
gcloud pubsub topics create $TOPIC
gcloud pubsub subscriptions create $SUB --topic $TOPIC
```

## Get JSON data
Simply run `python -m reddit.Main` from your Cloud Shell or local machine.

## Get images
Simply run `python -m reddit.ImageCrawler` from your Cloud Shell or local machine.

## Classify images
If you have an AutoML model, run `python2 automl_rest.py $img $project $model_id` from your Cloud Shell or local machine.

## Run DataFlow
Enable the required APIs for DataFlow, BigQuery, and Vision API and run the following code from your Google Cloud Shell:
```
python -m DataFlowReddit \
  --project $PROJECT \
  --runner DataflowRunner \
  --input gs://$BUCKET/json/picsreddit.json \
  --temp_location gs://$BUCKET/tmp/ \
  --bucket $BUCKET \
  --staging_location gs://$BUCKET/stg/ \
  --tmp /tmp/ \
  --useBigQuery true \
  --output reddit.posts \
  --imgOutput reddit.images \
  --requirements_file requirements.txt \
  --max_num_workers 24
```

## License
This project is licensed under the GNU GPLv3 License - see the [LICENSE](LICENSE) file for details.