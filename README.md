# Rel FAA Flight Model Project

## Overview

This codebase contains source code for the FAA Flight Model Project in Rel, developed to demonstrate the capabilities of the RAI system and use of the Rel language.
It contains files from RAI Console for loading the flight data and many example queries and
manipulations of the data.

## Getting Started

1. Acquire your OAuth Client ID and Secret from your account manager.
2. Put your credentials in a file called `config` inside a folder called `.rai` inside your home directory on your computer:
    ```ini
    [default]
    host = azure.relationalai.com
    client_id = <your client_id>
    client_secret = <your client secret>
    ```
4. Clone this repository (`git clone https://github.com/sswatson/faa-flights`) and open the folder in VS Code (`File` > `Open Folder...`).
5. Click the four-squares icon in the left sidebar to open the Extension Marketplace. Search for "RelationalAI" and click **Install**.
6. Use the RelationalAI icon in the left sidebar to open the RelationalAI panel. Create and select a database and an engine.
7. Load the data:
    - Open `data/load/load-simple-data.rel`, select the whole contents of the file (command-A) and do `shift+enter` to run.
    - Select `RelationalAI: Load CSV` from the command palette (command-shift-P) and enter the name `states_data`. Select the file `data/asset/states.csv`.
    - Select `RelationalAI: Load JSON` from the commmand pallete and enter the name `us_states_geojson`. Select the file `data/asset/us-states.json`.
8. Run example queries in `query` by selecting them one at a time and doing `shift+enter`. Outputs will appear in the RAI Output Viewer on the right.


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

## Guide

Check out the files in the `guide` directory for a guided tour of the code in this repo.