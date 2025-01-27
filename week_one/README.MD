# Homework Solutions (Week one)


## Question 1:

First, we need to run this command in the terminal:

```
docker run -it --entrypoint="/bin/bash" python:3.12.8
```

This command will open bash terminal at the docker image. Then, the following command is needed to run in that termianl.

```
pip --version
```

Then, we can see the following output as the result:

```
root@4d38473099ba:/# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```

So, `24.3.1` is the answer.

## Question 2:

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

In that `docker-compose.yaml` file, there are two services which have the hostname `db` and `pgadmin`. These two services can be commnicated between each other without specifying the network.

For `db` service, the container's internal port 5432 is map to the host machine's port 5433. So, `pgadmin` service can use the hostname `db` and port 5432  to connect to the postgres database.

## Question 3:

In order to run the query, we need to ingest the data to the postgres database first. At the same time, two services from docker need to get started. This can be done by using `docker compose up` command in the terminal.

Then, all the code cells inside the notebook `ingesting_data.ipynb` needs to be run.

After that, we can go to the web browser and type `http://localhost:8080/` to access the `pgadmin`.


```sql
SELECT
	DISTANCE_RANGE,
	count(1) AS CNT
FROM 
(
SELECT
	CASE 
		WHEN trip_distance <= 1 THEN '1M'
		WHEN trip_distance > 1 AND trip_distance <= 3 THEN '1M_3M'
		WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3M_7M'
		WHEN trip_distance > 7 AND trip_distance <= 10 THEN '7M_10M'
		ELSE 'Over_10M' END 
	AS DISTANCE_RANGE
FROM green_taxi_trips_201910 t 
) T1
GROUP BY 1 
```

Answer:

| DISTANCE_RANGE | CNT |
| :-------------- | :--------------:|
| 1M 	| 104838 |
| 1M_3M  |	199013 |
| 3M_7M  |	109645 |
| 7M_10M  |	27688 |
| Over_10M 	| 35202 |

## Question 4:

```sql
SELECT
	CAST(lpep_pickup_datetime AS DATE) AS lpep_pickup_date
FROM green_taxi_trips_201910
WHERE trip_distance = (SELECT MAX(trip_distance) FROM green_taxi_trips_201910) -- finding max trip distance
```

Answer: `2019-10-31`

## Question 5:

```sql
SELECT
	Z."Zone",
	SUM(total_amount) AS total_amount
FROM green_taxi_trips_201910 T
LEFT JOIN zones Z ON T."PULocationID" = Z."LocationID"
WHERE CAST(T.lpep_pickup_datetime AS DATE) = DATE '2019-10-18'
GROUP BY 1
HAVING SUM(total_amount) > 13000
ORDER BY total_amount DESC 
LIMIT 3 
```
Answer:
| Zone | total_amount |
| :---- | :----: |
| East Harlem North  |  18686.680000000088 |
| East Harlem South  |  16797.260000000068 |
| Morningside Heights  |  13029.79000000003 |

## Question 6:

```sql
SELECT
	DOZ."Zone" AS ZONE_NAME, T.tip_amount AS tip_amount
FROM green_taxi_trips_201910 T
LEFT JOIN zones PUZ ON T."PULocationID" = PUZ."LocationID"
LEFT JOIN zones DOZ ON T."DOLocationID" = DOZ."LocationID"
WHERE PUZ."Zone" = 'East Harlem North'
ORDER BY tip_amount DESC
LIMIT 1 
```
Answer:
| ZONE_NAME | tip_amount |
| :---- | :----: |
| JFK Airport | 87.3 |

## Question 7:

I think the answer should be

```
terraform init, terraform apply -auto-approve, terraform destroy
```



