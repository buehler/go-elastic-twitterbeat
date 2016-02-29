#Twitterbeat [![Build Status](https://travis-ci.org/buehler/go-elastic-twitterbeat.svg?branch=master)](https://travis-ci.org/buehler/go-elastic-twitterbeat)

TwitterBeat is a elastic [beat](https://www.elastic.co/products/beats) that fetches tweets from the Twitter Api (v1.1) and indexes them into elasticsearch. You can configure which screennames that should be fetched.

To use TwitterBeat, you need a valid Twitter Api Key (Consumer Key and stuff), an elasticsearch and Go. Or preferably a docker environment since I made a dockerized version of it.

##Elasticsearch template

To apply the elasticsearch index template:

```bash
curl -XPUT 'http://localhost:9200/_template/twitterbeat' -d@etc/twitterbeat.template.json
```

##Building

For now, I didn't manage to create a makefile :wink:. You need to build it from source, by hand. Download the source, install the dependencies with `glide` (make sure you have `GO15VENDOREXPERIMENT` set to 1) and then, run `go build` or `go install`.

I would recommend to use the docker image to run a containerized version of the twitterbeat.

##Run in docker

###Environment

| ENV variable      | Default value               | Description                 |
| ----------------- | --------------------------- | --------------------------  |
| `PERIOD`          | `60`                        | Refresh rate in seconds     |
| `SCREEN_NAMES`    | `["@smartive", "@elastic"]` | Screennames to fetch        |
| `ES_HOSTS`        | `["elasticsearch:9200"]`    | Hosts to index tweets to    |
| `CONSUMER_KEY`    | `null`                      | Twitter api consumer key    |
| `CONSUMER_SECRET` | `null`                      | Twitter api consumer secret |
| `ACCESS_KEY`      | `null`                      | Twitter api access key      |
| `ACCESS_SECRET`   | `null`                      | Twitter api access secret   |

###Volumes / mount

TwitterBeat has two volumes that can be mounted (both located in `/var/twitterbeat`):
- `config`
- `data`

Inside the config folder lies the `twitterbeat.yml`, the configuration of the beat.

In the data folder, the `twittermap.json` is stored, which is generated by the beat and keeps track, which tweet id is the newest fetched for each screen name. You might see content like this:

```json
{"@smartive":198759827398749873,"@elastic":87572876363772774}
```

If you delete the content of this file, twitterbeat will reindex the last 20 tweets of each screenname.

###Example

```bash
docker run -d \
       -e CONSUMER_KEY=<consumerKey> \
       -e CONSUMER_SECRET=<secret> \
       -e ACCESS_KEY=<accessKey> \
       -e ACCESS_SECRET=<secret> \
       --link elasticsearch:elasticsearch \
       buehler/go-elastic-twitterbeat
```

You can mount the data or config volume for custom configuration and persisting of the `twittermap.json` file.
```bash
docker run ... \
       -v $PWD/data:/var/twitterbeat/data \
       -v $PWD/twitterbeat.yml:/var/twitterbeat/config/twitterbeat.yml \
       buehler/go-elastic-twitterbeat
```
