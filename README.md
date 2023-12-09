# Technical Write-up

File architectire : 


## ETL Pipeline Schema

```mermaid

---
title: Pipeline Steps
---

flowchart TB
    
    subgraph Sources
        tripdatacsv["./bike-rental-starter-kit/data/JC-2016-{n}-citibike-tripdata.csv"]
        weathercsv["./bike-rental-starter-kit/data/newark_airport_2016.csv"]
    end

    tripdatacsv --> |"read_csv x12"| td_raw
    weathercsv --> |"read_csv x1"| wd_raw 

    td_raw -.-> |"Wrangling & Cleaning"| td_clean
    wd_raw -.-> |"Wrangling & Cleaning"| wd_clean

    subgraph Pandas
        td_raw["tripdata (raw)"]
        wd_raw["weather (raw)"]
        td_clean["tripdata (clean)"]
        wd_clean["weather (clean)"]
    end

    wd_clean --> |"Loading into"| fact_weather
    td_clean --> |"Loading into"| fact_trip

    subgraph PostgreSQL
        fact_trip
        fact_weather
        dim_bike
        dim_station
    end

    fact_trip --> dim_bike & dim_station
    fact_weather -.-> |Could enrich| dim_station

    fact_trip & fact_weather & dim_bike & dim_station --> Exposures

    subgraph Exposures
        Analytics_Dashboard
    end
    
```

## RDB Schema

```mermaid
---
title: RDB in PostGres
---
erDiagram
    dim_station {
        station_id INT UK "ID of the station | FK in fact_trip"
        station_name VARCHAR "Name of the station"
        station_latitude FLOAT "Latitude of the station"
        station_longitude FLOAT "Longitude of the station"
    }

    dim_bike {
        bike_id INT UK "Unique ID of the Bike | FK in fact_trip"
        number_of_use INT "Number of unique usage"
    }

    fact_trip {
        ride_id UUID UK "Unique ID of the trip"
        trip_duration INT "Duration of the trip in seconds"
        start_time DATETIME "Start time of the trip"
        stop_time DATETIME "Stop time of the trip"
        start_station_id INT FK "Foreign key referencing dim_station(station_id)"
        start_station_name VARCHAR "Name of the start station"
        start_station_latitude FLOAT FK "Latitude of the start station"
        start_station_longitude FLOAT FK "Longitude of the start station"
        end_station_id INT FK "Foreign key referencing dim_station(station_id)"
        end_station_name VARCHAR "Name of the end station"
        end_station_latitude FLOAT FK "Latitude of the end station"
        end_station_longitude FLOAT FK "Longitude of the end station"
        bike_id INT FK "Foreign key referencing dim_bike(bike_id)"
        user_type VARCHAR "Type of user (e.g., Subscriber, Customer)"
        birth_year INT "Birth year of the user"
        gender VARCHAR "Gender of the user"
    }

    fact_weather {
        station VARCHAR "Name of the weather station"
        date DATE PK "Date of the weather data"
        awnd FLOAT "Average daily wind speed (meters per second or miles per hour)"
        prcp FLOAT "Precipitation (mm or inches)"
        snow FLOAT "Snowfall (mm or inches)"
        snwd FLOAT "Snow depth (mm or inches)"
        tavg INT "Average temperature (Fahrenheit or Celsius)"
        tmax INT "Maximum temperature (Fahrenheit or Celsius)"
        tmin INT "Minimum temperature (Fahrenheit or Celsius)"
        wdf2 INT "Direction of fastest 2-minute wind (degrees)"
        wdf5 FLOAT "Direction of fastest 5-second wind (degrees)"
        wsf2 FLOAT "Fastest 2-minute wind speed (miles per hour or meters per second)"
        wsf5 FLOAT "Fastest 5-second wind speed (miles per hour or meters per second)"
        region VARCHAR "Geographical region"
        latitude FLOAT PK "Latitude of the weather station"
        longitude FLOAT PK "Longitude of the weather station"
    }

    fact_trip }o--|| dim_bike : "USING(bike_id)" 
    fact_trip }o--|| dim_station : "ON station_id = start_station_id OR end_station_id"
    fact_trip }o--|| fact_weather : "ON date = start_date OR end_date"


```

<!--
     "./bike-rental-starter-kit/data/JC-2016{n}-citibike-tripdata.csv" }o--|| "Pandas" : "Exploring & Cleaning"
    "Pandas" ||--|| "PostGres" : Loading
    "PostGres" ||--|| "fact_trip" : Managing

  "./bike-rental-starter-kit/data/newark_airport_2016.csv" }o--|| "Pandas" : "Exploring & Cleaning"
    "Pandas" ||--|| "PostGres" : Loading
    "PostGres" ||--|| "fact_weather" : Managing 
-->
