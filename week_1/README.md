# Week 1

## Docker and Postgres

Using WSL, Descktop Docker, VScode, Postgres


Additionally info:
  Green taxi data
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz

  Green taxi discription
https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_green.pdf

  Website TLC
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page



### Connecting localhost Postgres

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v dtc_postgres_volume_local:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

### Connecting pgcli to the localhost Postgres

pgcli -h localhost -p 5432 -u root -d ny_taxi


### Create volumes in Docker

docker volume create --name dtc_postgres_volume_local -d local

### Create network connection for docker compose up

docker network create pg-network

#### Connecting Postgres to the Docker

docker run -it \
-e POSTGRES_USER="root" \
-e POSTGRES_PASSWORD="root" \
-e POSTGRES_DB="ny_taxi" \
-v dtc_postgres_volume_local:/var/lib/postgresql/data \
-p 5432:5432 \
--network=pg-network \
--name pg-database \
postgres:13

#### Run pgAdmin

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin \
  dpage/pgadmin4
  
#### Create to the docker build

docker build -t taxi_ingest:v001 .

#### Data ingestion. Running locally

URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz"

docker run --network=pg-network taxi_ingest:v001 \
  --user=root \
  --password=root \
  --host=pg-database \
  --port=5432 \
  --db=ny_taxi \
  --table_name=green_taxi_trips \
  --url=${URL}

URL="https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv"

docker run --network=pg-network zone_script:v002 \
  --user=root \
  --password=root \
  --host=pg-database \
  --port=5432 \
  --db=ny_taxi \
  --table_name=zone_trips
  --url=${URL}

   
#### find, brings together and run container

docker compose up

