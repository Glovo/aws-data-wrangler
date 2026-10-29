<div align="center">

# 📚 aws-data-wrangler

**Glovo's fork of AWS Data Wrangler (pandas on AWS), published to Glovo Artifactory**

This repo **is** a fork of the open-source AWS Data Wrangler library (`awswrangler` 2.5.0). The library connects pandas to Athena, Glue, Redshift, Timestream, QuickSight, Chime, CloudWatch Logs, DynamoDB, EMR, Secrets Manager, PostgreSQL, MySQL, SQL Server and S3 (Parquet, CSV, JSON and Excel). Glovo added three things on top: a `connection_details` map for `wr.redshift.connect`, a Poetry build, and a publish to Glovo Artifactory on every tag. It **is not** a service, it is not kept in sync with upstream, and the upstream docs describe upstream releases, not this fork.

[![Python](https://img.shields.io/badge/Python-%5E3.8_%28Poetry%29-3776AB?logo=python&logoColor=white)](pyproject.toml)
[![pandas](https://img.shields.io/badge/pandas-%3E%3D1.1.0_%3C1.3.0-150458?logo=pandas&logoColor=white)](pyproject.toml)
[![PyArrow](https://img.shields.io/badge/PyArrow-%3E%3D2.0.0_%3C3.1.0-D22128?logo=apache&logoColor=white)](pyproject.toml)
[![Poetry](https://img.shields.io/badge/poetry--core-%3E%3D1.0.0-60A5FA?logo=poetry&logoColor=white)](pyproject.toml)
<br/>
[![Upstream](https://img.shields.io/badge/upstream-awswrangler_2.5.0-FF9900?logo=amazonwebservices&logoColor=white)](awswrangler/__metadata__.py)
[![Fork of](https://img.shields.io/badge/fork_of-aws%2Faws--sdk--pandas-181717?logo=github)](https://github.com/aws/aws-sdk-pandas)
[![Publish](https://img.shields.io/badge/publish-Artifactory_pypi--local-41BF47?logo=jfrog&logoColor=white)](.github/workflows/publish-package.yaml)
<br/>
[![Test files](https://img.shields.io/badge/test_files-28-blue)](tests)
[![mypy](https://img.shields.io/badge/mypy-strict-2A6DB2)](setup.cfg)
[![Code style: black](https://img.shields.io/badge/code_style-black-000000)](pyproject.toml)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue)](LICENSE.txt)

</div>

---

## Table of contents

- [Overview](#overview)
- [Build & local development](#build--local-development)
- [Testing](#testing)
- [Code quality](#code-quality)
- [CI/CD](#cicd)
- [Security & data handling](#security--data-handling)
- [Repository layout](#repository-layout)
- [Contributing & conventions](#contributing--conventions)
- [Glossary](#glossary)
- [Ownership & support](#ownership--support)

---

## Overview

`awswrangler` reads and writes **pandas DataFrames** against AWS data services, one call per operation: write a Parquet dataset to S3 and register it in the Glue Catalog, run an Athena query into a DataFrame, load or unload Redshift, write to Timestream, and so on. This fork is upstream `main` as of 2021-03-09 (version string **2.5.0** in [`__metadata__.py`](awswrangler/__metadata__.py)) plus 11 fork-only commits. When someone pushes a tag, CI builds the package with Poetry and uploads it to the `pypi-local` repository on Glovo Artifactory.

> [!NOTE]
> **Fork status (checked 2026-10-29):** the last commit on `main` is from 2021-03-11. `main` is 11 commits ahead of upstream [`aws/aws-sdk-pandas`](https://github.com/aws/aws-sdk-pandas) `main` and 1,629 commits behind it. Both CI test workflows are disabled in GitHub (see [CI/CD](#cicd)).

| | |
|---|---|
| **Language / build** | Python · Poetry ([`pyproject.toml`](pyproject.toml), what CI publishes) and setuptools ([`setup.py`](setup.py), upstream) |
| **Runtime** | Python `^3.8` for the Poetry package; `>=3.6, <3.10` in `setup.py` |
| **Owner** | No CODEOWNERS or catalog file; see [Ownership & support](#ownership--support) |

### What Glovo changed

| Change | Files | Commits |
|---|---|---|
| `connection_details` map: a third way to pass Redshift credentials to `wr.redshift.connect` | [`awswrangler/_databases.py`](awswrangler/_databases.py), [`awswrangler/redshift.py`](awswrangler/redshift.py) | `e5301f5` (2021-03-03), `35686da` |
| Poetry packaging: `[tool.poetry]` with name `awswrangler`, version `2.6.0`, plus a `src/awswrangler` symlink to `awswrangler/` | [`pyproject.toml`](pyproject.toml), [`src/awswrangler`](src/awswrangler) | `022b8ae`, `867d201` (2021-03-10/11) |
| Publish to Artifactory on every tag push | [`.github/workflows/publish-package.yaml`](.github/workflows/publish-package.yaml) | `fff81a8` (2021-03-11) |

### Install

> [!WARNING]
> The Poetry package has **the same name as upstream**: `awswrangler`. `pip install awswrangler` from public PyPI installs upstream, which has no `connection_details` option. The Glovo build is in the Artifactory repository that [`publish-package.yaml`](.github/workflows/publish-package.yaml) uploads to (`artifactory.glovoint.com`, repository `pypi-local`).

Upstream install commands:

| Source | Installation command |
|---|---|
| [PyPI](https://pypi.org/project/awswrangler/) (upstream) | `pip install awswrangler` |
| [Conda](https://anaconda.org/conda-forge/awswrangler) (upstream) | `conda install -c conda-forge awswrangler` |

> [!NOTE]
> **For platforms without PyArrow 3 support** (e.g. [EMR](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#emr-cluster), [Glue PySpark Job](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#aws-glue-pyspark-jobs), MWAA): `pip install pyarrow==2 awswrangler`

### Quick start

```python
import awswrangler as wr
import pandas as pd
from datetime import datetime

df = pd.DataFrame({"id": [1, 2], "value": ["foo", "boo"]})

# Storing data on Data Lake
wr.s3.to_parquet(
    df=df,
    path="s3://bucket/dataset/",
    dataset=True,
    database="my_db",
    table="my_table"
)

# Retrieving the data directly from Amazon S3
df = wr.s3.read_parquet("s3://bucket/dataset/", dataset=True)

# Retrieving the data from Amazon Athena
df = wr.athena.read_sql_query("SELECT * FROM my_table", database="my_db")

# Get a Redshift connection from Glue Catalog and retrieving data from Redshift Spectrum
con = wr.redshift.connect("my-glue-connection")
df = wr.redshift.read_sql_query("SELECT * FROM external_schema.my_table", con=con)
con.close()

# Amazon Timestream Write
df = pd.DataFrame({
    "time": [datetime.now(), datetime.now()],
    "my_dimension": ["foo", "boo"],
    "measure": [1.0, 1.1],
})
rejected_records = wr.timestream.write(df,
    database="sampleDB",
    table="sampleTable",
    time_col="time",
    measure_col="measure",
    dimensions_cols=["my_dimension"],
)

# Amazon Timestream Query
wr.timestream.query("""
SELECT time, measure_value::double, my_dimension
FROM "sampleDB"."sampleTable" ORDER BY time DESC LIMIT 3
""")
```

### Redshift `connection_details` (Glovo addition)

```python
con = wr.redshift.connect(
    connection_details={
        "kind": "redshift",      # must be "redshift"
        "host": "<host>",
        "port": "<port>",        # string, converted with int()
        "dbname": "<database>",
        "user": "<user>",
        "pass": "<password>",
    },
)
```

- **All six keys** are required, or you get `InvalidConnection`. A `kind` other than `redshift` raises `InvalidDatabaseType` ([`_databases.py`](awswrangler/_databases.py), [`redshift.py`](awswrangler/redshift.py)).
- **Precedence:** `connection` (Glue Catalog) wins over `secret_id` (Secrets Manager), which wins over `connection_details`. With the map, the `dbname` argument is ignored; the map's `dbname` is used.
- **Pass arguments by keyword.** `connection_details` was inserted as the 4th parameter of `connect()`, before `dbname`, so positional calls written for upstream shift by one.
- Only `wr.redshift.connect` accepts the map; `postgresql`, `mysql` and `sqlserver` `connect()` don't. The `connect()` docstring doesn't mention it yet.

### Logging

Enabling internal logging examples:

```python
import logging
logging.basicConfig(level=logging.INFO, format="[%(name)s][%(funcName)s] %(message)s")
logging.getLogger("awswrangler").setLevel(logging.DEBUG)
logging.getLogger("botocore.credentials").setLevel(logging.CRITICAL)
```

Into AWS Lambda:

```python
import logging
logging.getLogger("awswrangler").setLevel(logging.DEBUG)
```

<details><summary><b>Upstream documentation (v2.5.0), tutorials and API reference</b></summary>

These links point at the upstream docs for release 2.5.0, the release this fork is based on. The tutorials are the notebooks in this repo.

- [**Read The Docs**](https://aws-data-wrangler.readthedocs.io/)
- [**What is AWS Data Wrangler?**](https://aws-data-wrangler.readthedocs.io/en/2.5.0/what.html)
- [**Install**](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html)
  - [PyPi (pip)](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#pypi-pip)
  - [Conda](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#conda)
  - [AWS Lambda Layer](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#aws-lambda-layer)
  - [AWS Glue Python Shell Jobs](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#aws-glue-python-shell-jobs)
  - [AWS Glue PySpark Jobs](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#aws-glue-pyspark-jobs)
  - [Amazon SageMaker Notebook](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#amazon-sagemaker-notebook)
  - [Amazon SageMaker Notebook Lifecycle](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#amazon-sagemaker-notebook-lifecycle)
  - [EMR](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#emr)
  - [From source](https://aws-data-wrangler.readthedocs.io/en/2.5.0/install.html#from-source)
- [**Tutorials**](tutorials)
  - [001 - Introduction](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/001%20-%20Introduction.ipynb)
  - [002 - Sessions](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/002%20-%20Sessions.ipynb)
  - [003 - Amazon S3](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/003%20-%20Amazon%20S3.ipynb)
  - [004 - Parquet Datasets](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/004%20-%20Parquet%20Datasets.ipynb)
  - [005 - Glue Catalog](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/005%20-%20Glue%20Catalog.ipynb)
  - [006 - Amazon Athena](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/006%20-%20Amazon%20Athena.ipynb)
  - [007 - Redshift, MySQL, PostgreSQL, SQL Server](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/007%20-%20Redshift%2C%20MySQL%2C%20PostgreSQL%2C%20SQL%20Server.ipynb)
  - [008 - Redshift - Copy & Unload](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/008%20-%20Redshift%20-%20Copy%20%26%20Unload.ipynb)
  - [009 - Redshift - Append, Overwrite, Upsert](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/009%20-%20Redshift%20-%20Append%2C%20Overwrite%2C%20Upsert.ipynb)
  - [010 - Parquet Crawler](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/010%20-%20Parquet%20Crawler.ipynb)
  - [011 - CSV Datasets](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/011%20-%20CSV%20Datasets.ipynb)
  - [012 - CSV Crawler](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/012%20-%20CSV%20Crawler.ipynb)
  - [013 - Merging Datasets on S3](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/013%20-%20Merging%20Datasets%20on%20S3.ipynb)
  - [014 - Schema Evolution](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/014%20-%20Schema%20Evolution.ipynb)
  - [015 - EMR](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/015%20-%20EMR.ipynb)
  - [016 - EMR & Docker](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/016%20-%20EMR%20%26%20Docker.ipynb)
  - [017 - Partition Projection](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/017%20-%20Partition%20Projection.ipynb)
  - [018 - QuickSight](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/018%20-%20QuickSight.ipynb)
  - [019 - Athena Cache](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/019%20-%20Athena%20Cache.ipynb)
  - [020 - Spark Table Interoperability](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/020%20-%20Spark%20Table%20Interoperability.ipynb)
  - [021 - Global Configurations](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/021%20-%20Global%20Configurations.ipynb)
  - [022 - Writing Partitions Concurrently](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/022%20-%20Writing%20Partitions%20Concurrently.ipynb)
  - [023 - Flexible Partitions Filter](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/023%20-%20Flexible%20Partitions%20Filter.ipynb)
  - [024 - Athena Query Metadata](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/024%20-%20Athena%20Query%20Metadata.ipynb)
  - [025 - Redshift - Loading Parquet files with Spectrum](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/025%20-%20Redshift%20-%20Loading%20Parquet%20files%20with%20Spectrum.ipynb)
  - [026 - Amazon Timestream](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/026%20-%20Amazon%20Timestream.ipynb)
  - [027 - Amazon Timestream 2](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/027%20-%20Amazon%20Timestream%202.ipynb)
  - [028 - DynamoDB](https://github.com/Glovo/aws-data-wrangler/blob/main/tutorials/028%20-%20DynamoDB.ipynb)
- [**API Reference**](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html)
  - [Amazon S3](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-s3)
  - [AWS Glue Catalog](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#aws-glue-catalog)
  - [Amazon Athena](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-athena)
  - [Amazon Redshift](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-redshift)
  - [PostgreSQL](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#postgresql)
  - [MySQL](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#mysql)
  - [SQL Server](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#sqlserver)
  - [DynamoDB](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#dynamodb)
  - [Amazon Timestream](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-timestream)
  - [Amazon EMR](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-emr)
  - [Amazon CloudWatch Logs](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-cloudwatch-logs)
  - [Amazon Chime](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-chime)
  - [Amazon QuickSight](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#amazon-quicksight)
  - [AWS STS](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#aws-sts)
  - [AWS Secrets Manager](https://aws-data-wrangler.readthedocs.io/en/2.5.0/api.html#aws-secrets-manager)
- [**License**](LICENSE.txt)
- [**Contributing**](CONTRIBUTING.md)
- [**Legacy Docs** (pre-1.0.0)](https://aws-data-wrangler.readthedocs.io/en/0.3.3/)

</details>

<details><summary><b>Upstream community resources and adopters</b></summary>

Upstream collects these in its own README; it asks for additions by [pull request](https://github.com/awslabs/aws-data-wrangler/edit/main/README.md) with your resource or company name and your GitHub handle.

**Community resources**

- [Optimize Python ETL by extending Pandas with AWS Data Wrangler](https://aws.amazon.com/blogs/big-data/optimize-python-etl-by-extending-pandas-with-aws-data-wrangler/) [[igorborgest](https://github.com/igorborgest)]
- [Reading Parquet Files With AWS Lambda](https://aprakash.wordpress.com/2020/04/14/reading-parquet-files-with-aws-lambda/) [[anand086](https://github.com/anand086)]
- [Transform AWS CloudTrail data using AWS Data Wrangler](https://aprakash.wordpress.com/2020/09/17/transform-aws-cloudtrail-data-using-aws-data-wrangler/) [[anand086](https://github.com/anand086)]
- [Rename Glue Tables using AWS Data Wrangler](https://ananddatastories.com/rename-glue-tables-using-aws-data-wrangler/) [[anand086](https://github.com/anand086)]
- [Getting started on AWS Data Wrangler and Athena](https://medium.com/@dheerajsharmainampudi/getting-started-on-aws-data-wrangler-and-athena-7b446c834076) [[dheerajsharma21](https://github.com/dheerajsharma21)]
- [Simplifying Pandas integration with AWS data related services](https://medium.com/@bv_subhash/aws-data-wrangler-simplifying-pandas-integration-with-aws-data-related-services-2b3325c12188) [[bvsubhash](https://github.com/bvsubhash)]
- [Build an ETL pipeline using AWS S3, Glue and Athena](https://www.linkedin.com/pulse/build-etl-pipeline-using-aws-s3-glue-athena-data-wrangler-tom-reid/) [[taupirho](https://github.com/taupirho)]

**Who uses AWS Data Wrangler?** Upstream says knowing which companies use the library helps it prioritise the project internally.

- [Amazon](https://www.amazon.com/)
- [AWS](https://aws.amazon.com/)
- [Cepsa](https://cepsa.com) [[alvaropc](https://github.com/alvaropc)]
- [Cognitivo](https://www.cognitivo.ai/) [[msantino](https://github.com/msantino)]
- [Digio](https://www.digio.com.br/) [[afonsomy](https://github.com/afonsomy)]
- [DNX](https://www.dnx.solutions/) [[DNXLabs](https://github.com/DNXLabs)]
- [Funcional Health Tech](https://www.funcionalcorp.com.br/) [[webysther](https://github.com/webysther)]
- [Informa Markets](https://www.informamarkets.com/en/home.html) [[mateusmorato](http://github.com/mateusmorato)]
- [LINE TV](https://www.linetv.tw/) [[bryanyang0528](https://github.com/bryanyang0528)]
- [M4U](https://www.m4u.com.br/) [[Thiago-Dantas](https://github.com/Thiago-Dantas)]
- [nrd.io](https://nrd.io/) [[mrtns](https://github.com/mrtns)]
- [OKRA Technologies](https://okra.ai) [[JPFrancoia](https://github.com/JPFrancoia), [schot](https://github.com/schot)]
- [Pier](https://www.pier.digital/) [[flaviomax](https://github.com/flaviomax)]
- [Pismo](https://www.pismo.io/) [[msantino](https://github.com/msantino)]
- [ringDNA](https://www.ringdna.com/) [[msropp](https://github.com/msropp)]
- [Serasa Experian](https://www.serasaexperian.com.br/) [[andre-marcos-perez](https://github.com/andre-marcos-perez)]
- [Shipwell](https://shipwell.com/) [[zacharycarter](https://github.com/zacharycarter)]
- [strongDM](https://www.strongdm.com/) [[mrtns](https://github.com/mrtns)]
- [Thinkbumblebee](https://www.thinkbumblebee.com/) [[dheerajsharma21](https://github.com/dheerajsharma21)]
- [Zillow](https://www.zillow.com/) [[nicholas-miles](https://github.com/nicholas-miles)]

</details>

---

## Build & local development

> [!NOTE]
> **Prerequisites:** Linux or macOS · Python 3.7, 3.8 or 3.9 for development ([`CONTRIBUTING.md`](CONTRIBUTING.md); `setup.py` caps it at `<3.10`), 3.8+ for the Poetry build · AWS credentials (`AWS_PROFILE`, or `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`) and `AWS_DEFAULT_REGION` for anything beyond the mocked tests · Poetry, only to reproduce the published package.

```bash
python3 -m venv .venv && source .venv/bin/activate   # virtualenv for the project
pip install -r requirements-dev.txt                  # dev tools + editable install with the [sqlserver] extra
./validate.sh                                        # isort, black, mypy, flake8, pylint, pydocstyle, doc8
./fix.sh                                             # auto-format with isort + black
```

### Build the package

```bash
poetry install    # install [tool.poetry] dependencies
poetry build      # wheel + sdist in dist/; version from pyproject.toml (2.6.0)
```

This is what [`publish-package.yaml`](.github/workflows/publish-package.yaml) runs before `poetry publish`. The scripts in [`building/`](building) are upstream's. Run them from inside `building/`:

| Script | What it does |
|---|---|
| [`build-wheel.sh`](building/build-wheel.sh) | `python setup.py bdist_wheel`: the setuptools build, version from `__metadata__.py` |
| [`build-docs.sh`](building/build-docs.sh) | Sphinx HTML build of [`docs/`](docs), then doc8 |
| [`build-lambda-layers.sh`](building/build-lambda-layers.sh) | Lambda layer zips for Python 3.6, 3.7 and 3.8, built in `lambci/lambda` Docker images ([`lambda/Dockerfile`](building/lambda/Dockerfile)), compiling Apache Arrow 3.0.0 |
| [`publish.sh`](building/publish.sh) | Upstream release: egg, wheel and sdist with `python3.6`, then `twine upload dist/*` |

**Don't use `publish.sh` for Glovo releases.** It runs `twine upload` with no repository flag, which targets public PyPI. Glovo releases go through the tag-triggered workflow (see [CI/CD](#cicd)).

### Two build definitions

`setup.py` and `pyproject.toml` both describe the package, and they disagree. CI publishes the Poetry one.

| Setting | `setup.py` + `requirements.txt` (upstream) | `pyproject.toml` `[tool.poetry]` (published) |
|---|---|---|
| Version | `2.5.0`, read from [`__metadata__.py`](awswrangler/__metadata__.py) | `2.6.0` |
| Python | `>=3.6, <3.10` | `^3.8` |
| pg8000 | `>=1.16.0,<1.19.0` | `>=1.16.0,<1.18.0` |
| SQL Server extra | `sqlserver` → `pyodbc~=4.0.30` | none |

`wr.__version__` comes from `__metadata__.py`, so the 2.6.0 Poetry package still reports `2.5.0`.

---

## Testing

[`tests/`](tests) holds **28 Python test files**. Most of them call real AWS services. [`CONTRIBUTING.md`](CONTRIBUTING.md) offers three environments; you don't need to run everything to open a PR.

> [!WARNING]
> The data-lake and full environments **create real AWS resources that cost money**. The full stack runs Aurora MySQL, Aurora PostgreSQL and a single-node Redshift cluster, which cost money while running. Open the database security group **to your IP only**, never to the world.

| Environment | AWS resources | Setup |
|---|---|---|
| Mocked | None ([moto](https://github.com/spulec/moto)); fastest, mostly limited to S3 tests | none |
| Data lake | S3, Athena, Glue Catalog, KMS | `cloudformation/deploy-base.sh` |
| Full | Adds Redshift, Aurora PostgreSQL, Aurora MySQL, QuickSight and more | `deploy-base.sh` + `deploy-databases.sh` (about 15 minutes), then allow your IP |

```bash
pytest tests/test_moto.py::test_get_bucket_region_succeed   # one mocked test, no AWS needed
pytest -n 8 tests/test_moto.py                              # all mocked tests, 8 processes

export AWS_DEFAULT_REGION=ap-northeast-1                    # optional: a region you don't otherwise use
cd cloudformation
./deploy-base.sh                                            # data-lake stack aws-data-wrangler-base
./deploy-databases.sh                                       # full stack aws-data-wrangler-databases; prompts for a DB password
./security-group-databases-add-local-ip.sh                  # allow your current IP into the database security group
./security-group-databases-check.sh                         # confirm the rule
cd ..

pytest tests/test_athena_parquet.py::test_parquet_catalog   # one data-lake test
pytest -n 8 tests/test_athena*                              # all data-lake tests
pytest -n 8 tests/test_redshift.py                          # database tests; also test_postgresql.py, test_mysql.py, test_sqlserver.py
./test.sh                                                   # validate.sh + tox -e ALL + HTML coverage; needs QuickSight

./cloudformation/delete-base.sh                             # tear down the stacks afterwards
./cloudformation/delete-databases.sh
```

- Tests read the bucket, Glue database, KMS key and database endpoints from the outputs of the `aws-data-wrangler-base` and `aws-data-wrangler-databases` CloudFormation stacks ([`tests/_utils.py`](tests/_utils.py)).
- [`tox.ini`](tox.ini) runs `pytest -n 32 --timeout=300 tests` on py36–py39 (py39 adds coverage) and passes through `AWS_PROFILE`, `AWS_DEFAULT_REGION`, `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
- The full run needs **Amazon QuickSight activated** and your AWS user registered in it.
- `CONTRIBUTING.md` mentions `tests/test_db.py`, which no longer exists: database tests are split per engine (`test_redshift.py`, `test_postgresql.py`, `test_mysql.py`, `test_sqlserver.py`).
- No test covers the Glovo `connection_details` path.

<details><summary><b>Common install errors on macOS</b></summary>

From [`CONTRIBUTING_COMMON_ERRORS.md`](CONTRIBUTING_COMMON_ERRORS.md), which has the full error output:

| Error | Fix |
|---|---|
| `error: command 'clang' failed with exit status 1` while installing `python-Levenshtein` | `xcode-select --install` |
| `ImportError: dlopen(... pyodbc ...): Library not loaded: /usr/local/opt/unixodbc/lib/libodbc.2.dylib` | `brew install unixodbc` |

</details>

---

## Code quality

| Tool | Role |
|---|---|
| isort | Import order, 120 columns ([`pyproject.toml`](pyproject.toml) `[tool.isort]`) |
| black | Formatting, 120 columns, targets py36–py38 ([`pyproject.toml`](pyproject.toml) `[tool.black]`) |
| mypy | `strict = True`, `python_version = 3.6`; tests ignored ([`setup.cfg`](setup.cfg)) |
| flake8 | Lint, 120 columns, ignores E203 and W503 ([`setup.cfg`](setup.cfg)) |
| pylint | Lint with [`.pylintrc`](.pylintrc) |
| pydocstyle | Numpy docstring convention on `awswrangler/` |
| doc8 | RST lint on `docs/source`, ignores D005 and D002 |
| cfn-lint | CloudFormation templates, in CI and in the deploy scripts |

There are no git hooks. [`validate.sh`](validate.sh) runs the checks and [`fix.sh`](fix.sh) applies isort and black.

---

## CI/CD

| Workflow | Trigger | Effect | State (2026-10-29) |
|---|---|---|---|
| [`minimal-tests.yml`](.github/workflows/minimal-tests.yml) | Push or PR to `main` or `main-governed-tables` | Python 3.7: metadata, session, utils and moto tests (`AWS_DEFAULT_REGION=us-east-1`) | Disabled in GitHub |
| [`static-checking.yml`](.github/workflows/static-checking.yml) | Push or PR to `main` or `main-governed-tables` | cfn-lint, mypy, flake8, pylint, black, isort, pydocstyle, doc8 | Disabled in GitHub |
| [`publish-package.yaml`](.github/workflows/publish-package.yaml) | Push of any tag | Poetry build on Python 3.8, then `poetry publish` to Artifactory `pypi-local` with the `ARTIFACTORY_USERNAME` / `ARTIFACTORY_PASSWORD` secrets | Active |

> [!WARNING]
> **Publish Package fails as written.** Its "Install poetry" step downloads `get-poetry.py` from the `master` branch of the Poetry repo, which returns HTTP 404 (checked 2026-10-29). A tag push fails before anything is built.

> [!NOTE]
> Dependabot updates are configured in GitHub settings, not in a file in the repo. The 10 most recent runs (2026-01-22 to 2026-10-27) all failed.

### Releasing

- The publish job builds whatever version `pyproject.toml` declares (`2.6.0` on `main`). It runs on any tag. Glovo's tags `v2.6.0` to `v2.6.3` all point at the same commit, `fff81a8`.
- The workflow installs `poetry-dynamic-versioning`, but `pyproject.toml` has no `[tool.poetry-dynamic-versioning]` section.
- `bump2version major|minor|patch` (from `CONTRIBUTING.md`) rewrites the version string in the files listed in [`.bumpversion.cfg`](.bumpversion.cfg) (current `2.5.0`, including this README). That list doesn't include `pyproject.toml`. The config also sets `commit = False` and `tag = False`, so it creates no commit or tag, although `CONTRIBUTING.md` says it does.

---

## Security & data handling

- **Database credentials:** `wr.redshift.connect` gets them from a Glue Catalog connection (`connection`), a Secrets Manager secret (`secret_id`) or the Glovo `connection_details` map, checked in that order ([`_databases.py`](awswrangler/_databases.py)). The map holds the password in plain text under the key `pass`.
- **Test stacks:** [`deploy-databases.sh`](cloudformation/deploy-databases.sh) prompts for the database password and falls back to a default written in the script, so type your own. [`security-group-databases-add-local-ip.sh`](cloudformation/security-group-databases-add-local-ip.sh) opens all protocols and ports to your current IP (`/32`).
- **Reporting:** for security issues in the library, upstream asks you to use the [AWS vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/) and **not** to open a public GitHub issue ([`CONTRIBUTING.md`](CONTRIBUTING.md)). The repo has no `SECURITY.md`.

---

## Repository layout

```text
aws-data-wrangler/
├── awswrangler/                 # ⭐ the library: one module or package per AWS service
│   ├── athena/ catalog/ dynamodb/ quicksight/ s3/
│   ├── redshift.py              # Redshift I/O; connect() has the Glovo connection_details option
│   ├── _databases.py            # credential lookup shared by redshift, postgresql, mysql, sqlserver
│   └── __metadata__.py          # title, version (2.5.0), license
├── src/awswrangler              # symlink to ../awswrangler, added with the Poetry build
├── tests/                       # 28 Python test files; most need real AWS (see Testing)
├── tutorials/                   # numbered Jupyter tutorial notebooks
├── cloudformation/              # test stacks (base, databases) and their deploy/delete scripts
├── building/                    # upstream scripts: wheel, docs, Lambda layers, PyPI publish
├── docs/                        # Sphinx sources of the upstream docs site
├── .github/workflows/           # tests, static checks, Artifactory publish (see CI/CD)
├── pyproject.toml               # Poetry package (published) + black and isort config
├── setup.py, setup.cfg          # setuptools package (upstream) + flake8 and mypy config
├── requirements.txt             # runtime dependencies for pip installs
├── requirements-dev.txt         # dev tools + editable install
├── tox.ini                      # py36–py39 test matrix
└── validate.sh, fix.sh, test.sh # checks, auto-format, full test run
```

---

## Contributing & conventions

1. **Work against the latest `main`**, and check open and recently merged PRs first so you don't redo someone's work.
2. **Keep a PR to its change.** Don't reformat code you aren't changing; it hides the real diff.
3. **Run `./validate.sh`** and the tests your change touches before opening a PR. `./fix.sh` fixes formatting.
4. **120-character lines**, numpy-style docstrings, and strict mypy on `awswrangler/`.
5. **Discuss significant work** in an issue before starting it.
6. Glovo commits use `feat:`, `fix:` and `chore:` subject prefixes.
7. The [PR template](.github/PULL_REQUEST_TEMPLATE.md) asks you to confirm your contribution is made under the Apache 2.0 license.

<details><summary><b>More rules</b></summary>

- **VS Code:** `CONTRIBUTING.md` recommends black as the formatter, flake8 and mypy linting on, and pylint linting off.
- **Bug reports** (upstream): include a reproducible test case, the version you use, any modifications you made, and anything unusual about your environment.
- Upstream may ask for a Contributor License Agreement for larger changes. The project follows the Amazon Open Source Code of Conduct ([`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)).

</details>

There is no CODEOWNERS file, so GitHub doesn't request reviewers automatically.

---

## Glossary

| Term | Meaning |
|---|---|
| AWS Data Wrangler | The upstream open-source library (`awswrangler`). Open source, runs anywhere, and focused on code. Its repo is now [`aws/aws-sdk-pandas`](https://github.com/aws/aws-sdk-pandas). |
| Amazon SageMaker Data Wrangler | A SageMaker Studio feature with a similar name but a different purpose: specific to SageMaker Studio and focused on a visual interface. Not related to this library. |
| Glue Catalog connection | A named connection in the AWS Glue Data Catalog; `connect(connection=...)` reads host and credentials from it. |
| `connection_details` | Glovo's addition: a dict with `kind`, `host`, `port`, `dbname`, `user`, `pass` passed straight to `wr.redshift.connect`. |
| moto | Library that mocks AWS APIs; used by the mocked tests. |
| Lambda layer | A zip of the library and its dependencies for AWS Lambda, built by `building/build-lambda-layers.sh`. |
| PyArrow | Python bindings for Apache Arrow; the library uses it for Parquet I/O. |
| `pypi-local` | The Glovo Artifactory PyPI repository that the publish workflow uploads to. |

---

## Ownership & support

- **Team:** no CODEOWNERS or `catalog-info.yaml`. [`pyproject.toml`](pyproject.toml) lists the Machine Learning Platform Team (`tech.ml.platform@glovoapp.com`) as authors, and [`machine-learning-platform`](https://github.com/orgs/Glovo/teams/machine-learning-platform) is the GitHub team with admin access to the repo (checked 2026-10-29).
- **Upstream:** [`aws/aws-sdk-pandas`](https://github.com/aws/aws-sdk-pandas), an [AWS Professional Services](https://aws.amazon.com/professional-services/) open-source initiative · `aws-proserve-opensource@amazon.com`.
- **Security:** see [Security & data handling](#security--data-handling).
