# B2 Transcoder Web App

The [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) Transcoder Proof-of-Concept comprises a lightweight [Flask](https://palletsprojects.com/p/flask/) [worker app](https://github.com/Backblaze-B2-Samples/b2-transcoder-worker) and this web app implemented with [Django](https://www.djangoproject.com) and JavaScript. Together, they implement a simple video hosting site.

## User Experience

* Users upload videos from their browser via the web app.

* Once the video is uploaded, a JavaScript front end in the browser polls an API at the web app until the transcoded version is available.

* On receiving the video, the web app:

  * Uploads it to B2 via the [Backblaze S3 Compatible API](https://www.backblaze.com/b2/docs/s3_compatible_api.html).

  * Saves a record with the B2 object key to its local database.

  * POSTs a JSON notification to the worker app containing the video's B2 object key and a callback URL.

* The worker app:

  * Downloads the video from B2 to local storage.

  * Uses [`ffmpeg`](https://www.ffmpeg.org/download.html) to transcode the video.

  * Uploads the transcoded video to B2.

  * POSTs a JSON notification back to the web app containing the names of the original file and the transcoded version.

* The web app updates the video's database record with the name of the transcoded file.

* The next call from the JavaScript front end will return with the name of the transcoded video, signalling that the transcoding operation is complete. The browser shows the transcoded video, ready for viewing.

## Prerequisites

* [Python 3.9.2](https://www.python.org/downloads/release/python-392/) (other Python versions _may_ work) and `pip`
* You must install, configure and run the [B2 Transcoder Worker App](https://github.com/Backblaze-B2-Samples/b2-transcoder-worker). You may deploy both apps on the same host, listening on different ports, or on two different hosts. If the apps are running on different hosts, you must ensure that there is connectivity on the ports you are using.

## Installation

* Clone this repository and `cd` into the local repository directory.

* `pip install -r requirements`

## Configuration

* Create a `.env` file or set environment variables with your configuration:

    ```bash
    AWS_S3_REGION_NAME="<for example: us-west-001>"
    AWS_ACCESS_KEY_ID="<your B2 application key ID>"
    AWS_SECRET_ACCESS_KEY="<your B2 application key>"
    AWS_PRIVATE_BUCKET_NAME="<your private B2 bucket, for uploaded videos>"
    AWS_STORAGE_BUCKET_NAME="<your public B2 bucket, for static web assets>"
    TRANSCODER_WEBHOOK="<the API endpoint for the transcoder worker, e.g. http://1.2.3.4:5678/videos>"
    ```

* Run the usual commands to initialize a Django application:

    ```bash
    python manage.py migrate
    python manage.py createsuperuser
    python manage.py collectstatic
    ```

## Run the Web App

* To start the development server:

    ```bash
    python manage.py runserver
    ```

    You may provide the runserver command with the interface and port to which the web app should bind. For example, to listen on the standard HTTP port on all interfaces, you would use:

    ```bash
    python manage.py runserver 0.0.0.0:80
    ```

## Caveats

Note that this is a proof-of-concept system! To run a similar system in production, you would need to make several changes, including:

* Using a message queue such as [Apache Kafka](https://kafka.apache.org) or [RabbitMQ](https://www.rabbitmq.com) rather than HTTP POST notifications.
* Running the apps from a WSGI server such as [Green Unicorn](http://gunicorn.org/) or [Apache Web Server](https://httpd.apache.org) with [`mod_wsgi`](https://github.com/GrahamDumpleton/mod_wsgi).

Feel free to fork this repository and submit a pull request if you make an interesting change!

_Originally forked from the excellent [simple-s3-setup](https://github.com/sibtc/simple-s3-setup) by [sibtc](https://github.com/sibtc/)_.