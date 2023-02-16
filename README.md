<a href="https://cognite.com/">
    <img src="https://raw.githubusercontent.com/cognitedata/cognite-python-docs/master/img/cognite_logo.png" alt="Cognite logo" title="Cognite" align="right" height="80" />
</a>

# Cognite Python Replicator
[![build](https://webhooks.dev.cognite.ai/build/buildStatus/icon?job=github-builds/cognite-replicator/master)](https://jenkins.cognite.ai/job/github-builds/job/cognite-replicator/job/master/)
[![codecov](https://codecov.io/gh/cognitedata/cognite-replicator/branch/master/graph/badge.svg)](https://codecov.io/gh/cognitedata/cognite-replicator)
[![Documentation Status](https://readthedocs.com/projects/cognite-cognite-replicator/badge/?version=latest)](https://cognite-cognite-replicator.readthedocs-hosted.com/en/latest/)
[![PyPI version](https://badge.fury.io/py/cognite-replicator.svg)](https://pypi.org/project/cognite-replicator/)
[![tox](https://img.shields.io/badge/tox-3.6%2B-blue.svg)](https://www.python.org/downloads/release/python-366/)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/cognite-replicator)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

Cognite Replicator is a Python package for replicating data across Cognite Data Fusion (CDF) projects. This package is built on top of the Cognite Python SDK.
This component is Community content and not officially supported by Cognite. Bugs and changes will be fixed on a best effort basis. Feel free to open issues and pull requests, we will review them as soon as we can. 

Copyright 2023 Cognite AS

## Prerequisites
In order to start using the Replicator, you need:
- Python3 (>= 3.6)
- Credentials for both the source and destination projects: 
    - CLIENT_ID ("Client ID from Azure")
    - CLIENT_SECRET ("Client secret from Azure", only if using authentication via secret)
    - CLUSTER ("Name of CDF cluster")
    - TENANT_ID ("Tenant ID from Azure"
    - PROJECT ("Name of CDF project")

This is how you set the client secret as an environment variable on Mac OS and Linux:
```bash
$ export SOURCE_CLIENT_SECRET=<your source client secret>
$ export DEST_CLIENT_SECRET=<your destination client secret>
```

## Installation
The replicator is available on [PyPI](https://pypi.org/project/cognite-replicator/), and can also be executed .

To run it from command line, run:
```bash
pip install cognite-replicator
```

Alternatively, build and run it as a docker container. The image is avaible on [docker hub](https://hub.docker.com/r/cognite/cognite-replicator):
```bash
docker build -t cognite-replicator .
```

## Usage

### 1. Run with a configuration file as a standalone script

Create a configuration file based on the config/default.yml and update the values corresponding to your environment
If no file is specified then replicator will use config/default.yml.

via Python 

```bash
python -m cognite.replicator config/filepath.yml
```

or alternatively via docker

```bash
docker run -it cognite-replicator -e SOURCE_CLIENT_SECRET -e DEST_CLIENT_SECRET -v config/filepath.yml:/config.yml cognite-replicator /config.yml
```

### 2. Setup as Python library
#### 2.1 Without configuration file and interactive login 
It will copy everything from source to destination and use your own credentials to run the code, you need to have the right permissions to read on the source project and write on the destination project

```python
import os
import yaml
from cognite.client.credentials import OAuthInteractive
from cognite.client import CogniteClient, ClientConfig
from cognite.replicator import assets, events, files, time_series, datapoints, sequences, sequence_rows

# SOURCE
SOURCE_TENANT_ID = "48d5043c-cf70-4c49-881c-c638f5796997"
SOURCE_CLIENT_ID = "1b90ede3-271e-401b-81a0-a4d52bea3273"
SOURCE_PROJECT = "publicdata"
SOURCE_CLUSTER = "api"

# DESTINATION
DEST_TENANT_ID = "d4febcbc-db24-4823-bffd-92fd05b9c6bc"
DEST_CLIENT_ID = "189e8b95-f1ce-47d2-aa66-4c2fe3567f91"
DEST_PROJECT = "sa-team"
DEST_CLUSTER = "bluefield"

### Autogenerated variables
SOURCE_SCOPES = [f"https://{SOURCE_CLUSTER}.cognitedata.com/.default"]
SOURCE_BASE_URL = f"https://{SOURCE_CLUSTER}.cognitedata.com"
SOURCE_AUTHORITY_URL = f"https://login.microsoftonline.com/{SOURCE_TENANT_ID}"
DEST_SCOPES = [f"https://{DEST_CLUSTER}.cognitedata.com/.default"]
DEST_BASE_URL = f"https://{DEST_CLUSTER}.cognitedata.com"
DEST_AUTHORITY_URL = f"https://login.microsoftonline.com/{DEST_TENANT_ID}"

# Config
BATCH_SIZE = 10000  # this is the max size of a batch to be posted
NUM_THREADS = 10  # this is the max number of threads to be used
TIMEOUT = 90
PORT = 53000

SOURCE_CLIENT = CogniteClient(
    ClientConfig(
        credentials=OAuthInteractive(
            authority_url=SOURCE_AUTHORITY_URL,
            client_id=SOURCE_CLIENT_ID,
            scopes=SOURCE_SCOPES,
        ),
        project=SOURCE_PROJECT,
        base_url=SOURCE_BASE_URL,
        client_name="cognite-replicator-source",
    )
)
DEST_CLIENT = CogniteClient(
    ClientConfig(
        credentials=OAuthInteractive(
            authority_url=DEST_AUTHORITY_URL,
            client_id=DEST_CLIENT_ID,
            scopes=DEST_SCOPES,
        ),
        project=DEST_PROJECT,
        base_url=DEST_BASE_URL,
        client_name="cognite-replicator-destination",
    )
)

if __name__ == "__main__":  # this is necessary because threading

    #### Uncomment the resources you would like to copy
    assets.replicate(SOURCE_CLIENT, DEST_CLIENT)
    #events.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #files.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #time_series.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #datapoints.replicate(SOURCE_CLIENT, DEST_CLIENT)
    #sequences.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #sequence_rows.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
```

#### 2.2 Without configuration file and with client credentials authentication
It will copy everything from source to destination and use your own credentials to run the code, you need to have the right permissions to read on the source project and write on the destination project
(in the example below, the secrets are stored as environment variables)

```python
import os
from cognite.client.credentials import OAuthClientCredentials
from cognite.client import CogniteClient, ClientConfig
from cognite.replicator import assets, events, files, time_series, datapoints, sequences, sequence_rows

# SOURCE
SOURCE_TENANT_ID = "48d5043c-cf70-4c49-881c-c638f5796997"
SOURCE_CLIENT_ID = "1b90ede3-271e-401b-81a0-a4d52bea3273"
SOURCE_CLIENT_SECRET = os.environ.get("SOURCE_CLIENT_SECRET")
SOURCE_PROJECT = "publicdata"
SOURCE_CLUSTER = "api"

# DESTINATION
DEST_TENANT_ID = "d4febcbc-db24-4823-bffd-92fd05b9c6bc"
DEST_CLIENT_ID = "189e8b95-f1ce-47d2-aa66-4c2fe3567f91"
DEST_CLIENT_SECRET = os.environ.get("DEST_CLIENT_SECRET")
DEST_PROJECT = "sa-team"
DEST_CLUSTER = "bluefield"
### Autogenerated variables
SOURCE_SCOPES = [f"https://{SOURCE_CLUSTER}.cognitedata.com/.default"]
SOURCE_BASE_URL = f"https://{SOURCE_CLUSTER}.cognitedata.com"
SOURCE_TOKEN_URL = f"https://login.microsoftonline.com/{SOURCE_TENANT_ID}/oauth2/v2.0/token"
DEST_SCOPES = [f"https://{DEST_CLUSTER}.cognitedata.com/.default"]
DEST_BASE_URL = f"https://{DEST_CLUSTER}.cognitedata.com"
DEST_TOKEN_URL = f"https://login.microsoftonline.com/{DEST_TENANT_ID}/oauth2/v2.0/token"
COGNITE_CONFIG_FILE = "config/config.yml"
# Config
BATCH_SIZE = 10000  # this is the max size of a batch to be posted
NUM_THREADS = 10  # this is the max number of threads to be used
TIMEOUT = 90
PORT = 53000

SOURCE_CLIENT = CogniteClient(
    ClientConfig(
        credentials=OAuthClientCredentials(
            token_url=SOURCE_TOKEN_URL,
            client_id=SOURCE_CLIENT_ID,
            scopes=SOURCE_SCOPES,
            client_secret=SOURCE_CLIENT_SECRET,
        ),
        project=SOURCE_PROJECT,
        base_url=SOURCE_BASE_URL,
        client_name="cognite-replicator-source",
    )
)

DEST_CLIENT = CogniteClient(
    ClientConfig(
        credentials=OAuthClientCredentials(
            token_url=DEST_TOKEN_URL,
            client_id=DEST_CLIENT_ID,
            scopes=DEST_SCOPES,
            client_secret=DEST_CLIENT_SECRET,
        ),
        project=DEST_PROJECT,
        base_url=DEST_BASE_URL,
        client_name="cognite-replicator-destination",
    )
)

if __name__ == "__main__":  # this is necessary because threading

    #### Uncomment the resources you would like to copy
    assets.replicate(SOURCE_CLIENT, DEST_CLIENT)
    #events.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #files.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #time_series.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #datapoints.replicate(SOURCE_CLIENT, DEST_CLIENT)
    #sequences.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
    #sequence_rows.replicate(SOURCE_CLIENT, DEST_CLIENT, BATCH_SIZE, NUM_THREADS)
```

### 2.3 Alternative by having some elements of the configuration file as variable

Refer to [default configuration file](config/default.yml) or [example configuration file](config/example.yml) for all keys in the configuration file
Start with client creation from either step 2.1 or 2.2

```python

if __name__ == "__main__":  # this is necessary because threading
    config = {
        "timeseries_external_ids": ["pi:160670", "pi:160623"],
        "datapoints_start": "100d-ago",
        "datapoints_end": "now",
    }
    time_series.replicate(
        client_src=SOURCE_CLIENT,
        client_dst=DEST_CLIENT,
        batch_size=BATCH_SIZE,
        num_threads=NUM_THREADS,
        config=config,
    )
    datapoints.replicate(
        client_src=SOURCE_CLIENT,
        client_dst=DEST_CLIENT,
        external_ids=config.get("timeseries_external_ids"),
        start=config.get("datapoints_start"),
        end=config.get("datapoints_end"),
    )
```

### 3. With configuration file
It will use the configuration file to determine what will be copied
In this case, no need to create the client, it will be created based on what is in the configuration file

```python
import yaml
from cognite.replicator.__main__ import main
import os

if __name__ == "__main__":  # this is necessary because threading
    COGNITE_CONFIG_FILE = yaml.safe_load("config/config.yml")
    os.environ["COGNITE_CONFIG_FILE"] = COGNITE_CONFIG_FILE
    main()
```


## Development

Change the version in the files
- [_version.py](cognite/replicator/_version.py#L1 "Version in code")
- [cd.yml](.github/workflows/cd.yml#L30 "Continuous deployment yaml file")
- [pyproject.toml](pyproject.toml#L3 "Poetry configuration")


## Changelog
Wondering about upcoming or previous changes? Take a look at the [CHANGELOG](https://github.com/cognitedata/cognite-replicator/blob/master/CHANGELOG.md).

## Contributing
Want to contribute? Check out [CONTRIBUTING](https://github.com/cognitedata/cognite-replicator/blob/master/CONTRIBUTING.md).
