# OpenFaas Fluentd

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)

A [Fluentd](https://www.fluentd.org/) openfaas template.

Templates available in this repository:
- fluentd

## Downloading the template

```
$ faas template pull https://github.com/terradue/openfaas-fluentd
```

## Using the python3-fastapi-conda templates

Create a new function

```
$ faas new --lang fluentd <fn-name>
```

Update the `td-agent.conf` file with a fluentd configuration that must use the [in_http] plugin allowing you to send events through HTTP requests. 


Build, push, and deploy

```
$ faas up -f <fn-name>.yml
```

Set your OpenFaaS gateway URL. For example:

```
$ OPENFAAS_URL=http://127.0.0.1:8080
```

Test the new function

```
$ curl -i $OPENFAAS_URL/function/<fn-name>
```

## Usage 

### Python

Below an example of a logger using HTTP POST requests to the openfaas fluentd function:

```python
import json
import logging
import requests, platform 
import datetime
from pythonjsonlogger import jsonlogger

class CustomHttpHandler(logging.Handler):
    def __init__(self, url: str, silent: bool = True):
        '''
        Initializes the custom http handler
        Parameters:
            url (str): The URL that the logs will be sent to
            token (str): The Authorization token being used
            silent (bool): If False the http response and logs will be sent 
                           to STDOUT for debug
        '''
        self.url = url
        self.silent = silent


        super().__init__()

    def emit(self, record):
        '''
        This function gets called when a log event gets emitted. It recieves a
        record, formats it and sends it to the url
        Parameters:
            record: a log record
        '''
        logEntry = self.format(record)
        response = requests.post(self.url, data=logEntry)

        if not self.silent:
            print(logEntry)
            print(response.status_code, response.reason)

class HostnameFilter(logging.Filter):
    hostname = platform.node()

    def filter(self, record):
        record.hostname = HostnameFilter.hostname
        return True


logger = logging.getLogger("name")
logger.setLevel(logging.DEBUG)

formatter = jsonlogger.JsonFormatter("%(asctime)s%(hostname)s %(processName)s %(name)s %(levelname)s %(message)s", "%Y-%m-%dT%H:%M:%SZ")

httpHandler = CustomHttpHandler(
    url="<function URL>",
    silent=silent
)

httpHandler.addFilter(HostnameFilter())

httpHandler.setLevel(logging.DEBUG)

httpHandler.setFormatter(formatter)

logger.addHandler(httpHandler)

logger.info('{"message": "a message"}')
```


