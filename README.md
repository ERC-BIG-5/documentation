The BIG-5 project is structured into several python packages, with different responsibilities.

The packages are managed with **[uv](https://docs.astral.sh/uv/)**, which is highly recommended to install project dependencies.

## Main repositories / packages

``` mermaid
graph

platform-clients -- dump data--> databases
databases <-- reade/write --> pipelines
pipelines <-- "create LS task files,read annotation results" --> labelstudio-tools
labelstudio-tools <-- api --> Labelstudio[[Labelstudio instance on our VM]]
```

### platform-clients

This package includes packages and implementations to query different social media platforms (SM). It allows the creation of collection-tasks, in the form of json files, which contain information about how to query different social media platforms.

Through integration (as a dependency) of the *databases* package, all collection-tasks and resulting posts (results of api requests, not including media files) are stored in sqlite databases, ready to be processed further (by the pipeline).

Based on the access to different SM, the user/developer has to install extra dependencies, which then allow them to run collection-tasks for the corresponding SM.

For testing of the package, access to YouTube with an API-key can configured here: https://console.cloud.google.com/welcome (with any google account).

Repos:
https://gitlab.bsc.es/rsoleyma/platform_clients
https://github.com/ERC-BIG-5/platform-clients.git

This repo is already more developer/user friendly than the others. Containing a detailed readme on how to setup data collection.

### databases

This is the repository that manages collection-tasks and collected data. It uses SqlAlchemy (see below) to store data in sqlite databases. It uses several SM independent tables to store data for collection-tasks, posts, users, comments (tho only collection-tasks, posts are developed in platform-clients package so far).

Repos:
https://gitlab.bsc.es/rsoleyma/big5_databases
https://github.com/ERC-BIG-5/databases

### pipelines

The package, is for flexible data processing. It is the glue, that batch processes database entries, filtering or running special tasks on them.

It allows to define process-methods (python scripts), with configuration models, which can be part of the pipelines packages or sit anywhere on the executing system. A configuration file, can specify a set of process-methods with task specific configurations and flows, which execute a set of process-methods in a order.
Examples flows:
**sample downloaded posts and download mediafiles**
1- iterate trough a databases table of posts
2-Select post for a 10 hour timewindow within a range of 2 years
3-Iterate through those posts, and pick the first one that contains media files
4-download those media-files
5-store information about those media files back into the database for that post (file-locations or errors).

**create Labelstudio tasks**
1- iterate trough a databases table of posts
2-Select post for a 10 hour timewindow within a range of 2 years
3-mark one random post as designated for labeling
4-create json file, that can be imported to Labelstudio
5-store selection, back into the database

repo:
https://gitlab.bsc.es/rsoleyma/datapipeline

### labelstudio-tools

This package handles the interaction to [Labelstudio](https://labelstud.io/). It uses the API of an Labelstudio (LS) instance, to create LS projects, tasks and collects and analyses the annotation results.

repos:
https://github.com/ERC-BIG-5/labelstudio-tools

### Python packages

These 3rd party python packages are essential for one or multiple of the BIG-5 packages.

#### Pydantic

> [Pydantic](https://docs.pydantic.dev/latest/) is the most widely used data validation library for Python.
> Fast and extensible, Pydantic plays nicely with your linters/IDE/brain. Define how data should be in pure, canonical Python 3.9+; validate it with Pydantic.

We are using pydantic all over the place, in order to avoid moving python dictionaries around. This is for many reasons very useful. It guarantees the data has the shape we want and takes a lot of overhead from the developer, by not requiring them to remember dict keys or search around in the code, how keys are called. In addition we also use pydantic-settings, in order to load environment variables.

#### Sqlalchemy

> [The Python SQL](https://www.sqlalchemy.org/) Toolkit and Object Relational Mapper
> SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that gives application developers the full power and flexibility of SQL.

We use Sqlalchemy for managing our databases. So instead of writing sql queries, sqlalchemy abstracts that awat and we can just write python code and deal with python objects, instead that represent database rows.

#### Typer

> [Typer](https://typer.tiangolo.com/) is a library for building CLI applications that users will **love using** and developers will **love creating**. Based on Python type hints.
> It's also a command line tool to run scripts, automatically converting them to CLI applications.

#### python-project-tools

A [helper package](https://github.com/transfluxus/python-project-tools) developed by Ramin. We are using the project-logger, file-reader, and project-directory modules from this package.
