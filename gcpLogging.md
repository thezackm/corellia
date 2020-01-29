# Stackdriver Logs to New Relic Logs via Logstash

- Overview
  * [Reference Docs](#reference-docs)
  * [Enable GCP APIs](#enable-gcp-apis)
  * [Create GCP Service Account](#create-gcp-service-account)
  * [Create GCP Pub/Sub Topic and Subscription](#create-gcp-pub-sub-topic-and-subscription)
  * [Create Stackdriver Logging sink](#create-stackdriver-logging-sink)
  * [Create Logstash VM in GCE](#create-logstash-vm-in-gce)
  * [Install and Configure Logstash](#install-and-configure-logstash)
  * [Test Connectivity](#test-connectivity)

## Reference Docs
[Logstash Plugin](https://github.com/logstash-plugins/logstash-input-google_pubsub)

[Stackdriver Logs Exports](https://cloud.google.com/logging/docs/export/)

[GCP Demo for Elastic Search](https://cloud.google.com/community/tutorials/exporting-stackdriver-elasticcloud)

[New Relic Logstash](https://docs.newrelic.com/docs/logs/enable-logs/enable-logs/logstash-plugin-logs)

[Stackdriver Monitored Resource Types](https://cloud.google.com/monitoring/api/resources)

## Enable GCP APIs
* Compute Engine
* Cloud Pub/Sub
* IAM
* Stackdriver
* Cloud Functions (Only needed for testing)

`gcloud services enable compute.googleapis.com pubsub.googleapis.com iam.googleapis.com stackdriver.googleapis.com cloudfunctions.googleapis.com`

## Create GCP Service Account
Create a Service Account to use with least-privileged access for running your VM

`gcloud iam service-accounts create logstash --display-name="Stackdriver to Logstash"`

Add the Pub/Sub Subscriber role to the new Service Account

`gcloud projects add-iam-policy-binding <projectName> --member serviceAccount:logstash@<projectName>.iam.gserviceaccount.com --role roles/pubsub.subscriber`

## Create GCP Pub/Sub Topic and Subscription
Create a Pub/Sub Topic where Stackdriver will send events to be picked up

`gcloud pubsub topics create stackdriver-topic`

Create a Pub/Sub Subscription where Logstash will pull events from

`gcloud pubsub subscriptions create logstash-sub --topic=stackdriver-topic --topic-project=<projectName>`

## Create Stackdriver Logging sink
Create a log sink for Stackdriver to export events to the Pub/Sub Topic

`gcloud logging sinks create logstash-sink pubsub.googleapis.com/projects/<projectName>/topics/stackdriver-topic --log-filter='<optionalFilter>'`

This will generate output similar to below, which notes that you need to add permissions to the service account used by Stackdriver in order for it to publish to the Pub/Sub Topic:

```
Created [https://logging.googleapis.com/v2/projects/<projectName>/sinks/logstash-sink].
Please remember to grant `serviceAccount:p280205150370-236177@gcp-sa-logging.iam.gserviceaccount.com` the Pub/Sub Publisher role on the topic.
```
Add the required role to the service account:

`gcloud pubsub topics add-iam-policy-binding stackdriver-topic --member serviceAccount:p280205150370-236177@gcp-sa-logging.iam.gserviceaccount.com --role roles/pubsub.publisher`

## Create Logstash VM in GCE
Build an Ubuntu VM (named "logstash") to run our Logstash installation

```
gcloud compute --project=<projectName> instances create logstash \
--zone=us-central1-a \
--machine-type=n1-standard-1 \
--subnet=default \
--service-account=logstash@<projectName>.iam.gserviceaccount.com \
--scopes="https://www.googleapis.com/auth/cloud-platform" \
--image-family=ubuntu-1804-lts \
--image-project=ubuntu-os-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=logstash
```

## Install and Configure Logstash
SSH to your newly created VM

`gcloud compute ssh logstash --project '<projectName>' --zone '<vmZone>'`

Run typical updates, install OpenJDK, and reboot the VM

```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt -y install openjdk-8-jre-headless
echo "export JAVA_HOME=\"/usr/lib/jvm/java-8-openjdk-amd64\"" >> ~/.profile
sudo reboot
```

SSH back into the VM and Install Logstash from the Elastic repo
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update
sudo apt-get install logstash
```

Install the Logstash Plugins for Cloud Pub/Sub and New Relic

```
cd /usr/share/logstash
sudo -u root sudo -u logstash bin/logstash-plugin install logstash-input-google_pubsub logstash-output-newrelic
```

Build new config file and add the contents below: `sudo vim /etc/logstash/conf.d/logstash.conf`

<details>
<summary>Example for Testing</summary>
<p>

```
input
{
    google_pubsub {
        project_id => "<projectName>"
        topic => "stackdriver-topic"
        subscription => "logstash-sub"
        include_metadata => true
        codec => "json"
    }
    # optional, but helpful to generate the ES index and test the plumbing
    heartbeat {
        interval => 10
        type => "heartbeat"
    }
}
filter {
    # don't modify logstash heartbeat events
    if [type] != "heartbeat" {
        mutate {
            add_field => { "messageId" => "%{[@metadata][pubsub_message][messageId]}" }
        }
    }
}
output
{
    newrelic {
      license_key => "<licenseKey>"
    }
}
```

</p>
</details> 

_NOTE: In production, you'll want to remove the Heartbeat from the config file to save costs and reduce noise_

<details>
<summary>Example for Prod</summary>
<p>

```
input
{
    google_pubsub {
        project_id => "<projectName>"
        topic => "stackdriver-topic"
        subscription => "logstash-sub"
        include_metadata => true
        codec => "json"
    }
}
filter {
    mutate {
        add_field => { "messageId" => "%{[@metadata][pubsub_message][messageId]}" }
    }
}
output
{
    newrelic {
      license_key => "<licenseKey>"
    }
}
```

</p>
</details> 

Start Logstash: `sudo service logstash start`

Tail startup log for errors: `sudo tail -f /var/log/syslog`

## Test Connectivity

If necessary, you can [Build a small Cloud Function](https://cloud.google.com/functions/docs/quickstart-console) to test the Pub/Sub settings.

<details>
<summary>Cloud Function source code: helloWorld</summary>
<p>

```javascript

/**
 * Responds to any HTTP request.
 *
 * @param {!express:Request} req HTTP request context.
 * @param {!express:Response} res HTTP response context.
 */
exports.helloWorld = (req, res) => {
  let message = req.query.message || req.body.message || 'Hello World!';
  console.log('I am a log entry!');
  console.error('I am an error!');  
  res.status(200).send(message);
};

```

</p>
</details> 