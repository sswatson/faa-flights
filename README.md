# Rel FAA Flight Model Project

## Overview

This codebase contains source code for the FAA Flight Model Project in Rel, developed to demonstrate the capabilities of the RAI system and use of the Rel language.
It contains files from RAI Console for loading the flight data and many example queries and
manipulations of the data.

## Getting Started

1. Acquire your OAuth Client ID and Secret from your account manager.
2. Put your credentials in a file called `config` inside a folder called `.rai` inside your home directory on your computer.
3. Clone this repository (`git clone https://github.com/sswatson/faa-flights`) and open the folder in VS Code (`File` > `Open Folder...`).
4. Click the four-squares icon in the left sidebar to open the Extension Marketplace. Search for "RelationalAI" and click **Install**.
5. Use the RelationalAI icon in the left sidebar to open the RelationalAI panel. Create and select a database and an engine.
6. Load the data:
    - Open `query/write/load-sample-data.rel`, select the whole contents of the file (command-A) and do `shift+enter` to run.
    - Select `RelationalAI: Load CSV` from the command palette (command-shift-P) and enter the name `states_data`. Select the file `data/states.csv`.
    - Select `RelationalAI: Load JSON` from the commmand pallete and enter the name `us_states_geojson`. Select the file `data/us-states.json`.
7. Run example queries in `query/read` by selecting them one at a time and doing `shift+enter`. Outputs will appear in a panel on the right.

## Files

- Loading data:
  - `rel/load-data.rel`: Queries to install complete dataset
  - `rel/load-sample-data.rel`: Queries to install sample dataset
- Model source code: `rel/faa-flight/`
- Example queries for using and exploring the data/model: `rel/queries`
  - `rel/queries/rel-by-example` contains example queries based on [Malloy by Example](https://looker-open-source.github.io/malloy/documentation/index).

## Data Sources

The flight data is public, located in several places:
- In [this AWS Bucket](https://s3.console.aws.amazon.com/s3/buckets/malloy-faa-flights-data) from RAI (uploaded from BigQuery)
- In BigQuery from Malloy:
  - `malloy-data.faa.flights`
  - `malloy-data.faa.airports`
  - `malloy-data.faa.aircraft`
  - `malloy-data.faa.aircraft_models`
  - `malloy-data.faa.carriers`

This project is inspired by the [documentation from
Malloy.](https://looker-open-source.github.io/malloy/documentation/index.html)
