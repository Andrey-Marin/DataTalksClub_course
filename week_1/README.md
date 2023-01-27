# Week 1


Docker is a tool that delivers software in containers, which are isolated from each other, and contain all code and dependencies required to run some service or task (e.g., data pipeline).

Docker has several advantages:
1. It is easy to reproduce data pipelines in different environments.
2. We can run local experiments and local tests, such as integration tests.
3. It is useful to perform integration tests under CI/CD.
4. We can deploy pipelines in the cloud (e.g., AWS Batch and Kubernetes jobs).
5. We can process data using Serverless services (e.g., AWS Lambda).

#### Additionally info:

Using tools in Windows: WSL, Descktop Docker, VScode, Postgres.

Green taxi data:
```
https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz
```

Taxi zone lookup:
```
https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Green taxi discription:

[https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_green.pdf](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_green.pdf)


Website TLC:

[https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

## Connecting to the database postgres in terminal as follow.

**Step 1:** create volumes in Docker and create and run an empty database postgres.

    docker volume create --name dtc_postgres_volume_local -d local

    docker run -it \
      -e POSTGRES_USER="root" \
      -e POSTGRES_PASSWORD="root" \
      -e POSTGRES_DB="ny_taxi" \
      -v dtc_postgres_volume_local:/var/lib/postgresql/data 
      -p 5432:5432 \
      postgres:13

**Step 2:** we can acces to the empty database postgres.

    pip instal pgcli

    pgcli -h localhost -p 5432 -u root -d ny_taxi

## Connecting to the pgadmin. 

**Step 1:** create network connection.

    docker network create pg-network
    
**Step 2:** create and run pastgres with network.

    docker run -it \
      -e POSTGRES_USER="root" \
      -e POSTGRES_PASSWORD="root" \
      -e POSTGRES_DB="ny_taxi" \
      -v dtc_postgres_volume_local:/var/lib/postgresql/data \
      -p 5432:5432 \
      --network=pg-network \
      --name pg-database \
      postgres:13

**Step 3:** create and run pgAdmin with network.

    docker run -it \
      -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
      -e PGADMIN_DEFAULT_PASSWORD="root" \
      -p 8080:80 \
      --network=pg-network \
      --name pgadmin \
      dpage/pgadmin4
  
Now we have linked postgres and pgadmin.

## Loading data into the database

**Step 1:** create image (loading taxi data) with [Dockerfile](./Dockerfile_1) using [ingest_data.py](./ingest_data.py) and run.

    docker build -t taxi_ingest:v001 .

    URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz"

    docker run --network=pg-network taxi_ingest:v001 \
      --user=root \
      --password=root \
      --host=pg-database \
      --port=5432 \
      --db=ny_taxi \
      --table_name=green_taxi_trips \
      --url=${URL}

**Step 2:** create image (loading taxi zone data) with [Dockerfile](./Dockerfile_2) using [zone_script.py](./zone_script.py) and run.

    docker build -t zone_script:v001 .

    URL="https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv"

    docker run --network=pg-network zone_script:v001 \
      --user=root \
      --password=root \
      --host=pg-database \
      --port=5432 \
      --db=ny_taxi \
      --table_name=zone_trips
      --url=${URL}

Now, we are able to create the server using pgadmin's interface with data green taxi and taxi zone.  

## Multi-container docker applications.

Docker compose allows us to run multi-container docker applications, by specifying a number of services that must be run together in a single YAML file. For more information, see [docker docs](https://docs.docker.com/compose/).

**Step 1:** create [docker-compose.yaml](./docker-compose.yaml).
    
**Step 2:** launch docker compose.

    docker compose up

Now, we can using pgadmin's interface.
