# Elastic stack (ELK) on Docker

[![Elastic Stack version](https://img.shields.io/badge/Elastic%20Stack-8.7.1-00bfb3?style=flat&logo=elastic-stack)](https://www.elastic.co/blog/category/releases)
[![Build Status](https://github.com/deviantony/docker-elk/workflows/CI/badge.svg?branch=main)](https://github.com/deviantony/docker-elk/actions?query=workflow%3ACI+branch%3Amain)
[![Join the chat](https://badges.gitter.im/Join%20Chat.svg)](https://app.gitter.im/#/room/#deviantony_docker-elk:gitter.im)

Run the latest version of the [Elastic stack][elk-stack] with Docker and Docker Compose.




![Animated demo](https://user-images.githubusercontent.com/3299086/155972072-0c89d6db-707a-47a1-818b-5f976565f95a.gif)




### Host setup

* [Docker Engine][docker-install] version **18.06.0** or newer
* [Docker Compose][compose-install] version **1.28.0** or newer (including [Compose V2][compose-v2])
* 1.5 GB of RAM

> **Note**  
> Especially on Linux, make sure your user has the [required permissions][linux-postinstall] to interact with the Docker
> daemon.

By default, the stack exposes the following ports:

* 5044: Logstash Beats input
* 50000: Logstash TCP input
* 9600: Logstash monitoring API
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

### Bringing up the stack

Clone this repository onto the Docker host that will run the stack with the command below:

```sh
git clone https://github.com/deviantony/docker-elk.git
```

Then, initialize the Elasticsearch users and groups required by docker-elk by executing the command:

```sh
docker-compose up setup
```

If everything went well and the setup completed without error, start the other stack components:

```sh
docker-compose up
```

### Integrating logs from django application.


1. Install the following module for logstash integration so that we can send logs to ELK stack.
`pip3 install python-logstash`
2. Open the settings.py file in django application, update the Logger settings as following
```
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'simple': {
            'format': '[%(asctime)s] %(levelname)s %(message)s',
            'datefmt':'%Y-%m-%dT%H:%M:%S'
        }
    },
    'handlers': {
        'file': {
            'class': 'logging.FileHandler',
            'filename': 'warning.log',
            'level': 'WARNING',
            'formatter': 'simple'
        },
        'logstash': {
            'level': 'WARNING',
            'class': 'logstash.TCPLogstashHandler',
            'host': 'localhost',
            'port': 50000,
            'version': 1,
            'message_type': 'django',
            'tags': ['django'],
        },
    },
    'loggers': {
        '': {
            'handlers': ['file', 'logstash'],
            'level': 'WARNING',
            'propagate': True,
        },
    },
}
```

  
3. Run any python application with following command `python3 manage.py runserver 8500`
  


** Notes **
1. All the logs that are level -  WARNING will be sent to logstash.
2. Logstash.conf file contains the details about the port and format of data it expects from django.
```
  tcp {
		port => 50000
		codec => json
	}
```
3. Make sure all the docker conainer of ELK stash are up and helathy.
4. We can debug the logs in kibana with dev tools.
   4.1 Open dev tools in kibana running at localhost:5601
   4.2 Run the following query in dev tool
   `GET /_cat/indices`
   This will fetch all the indexes that are being rendered by elasticsearch after being sent from logstash.
   4.3 If index-name of logs are `.ds-logs-generic-default-2023.05.04-000001`, then results of index can be seen with following query
   
   `GET /.ds-logs-generic-default-2023.05.04-000001/_search?pretty`