# Docker

This describes the steps to getting a docker image built and run that contains a Linux alpine distribution with postgresql, timescaledb, postgis, and loads the shuttle DB schema and all the CSV data.

## Steps

* Stop local instances of postgres (if necessary); e.g. on MacOS:
  ```
  C02RP8FEG8WP:shuttle_db aleung181$ brew services stop postgresql
  ```
  
* Install Docker and run it
* clone shuttle_analysis:

  ```
  C02RP8FEG8WP:sfmta aleung181$ git clone https://github.com/SFMTA/shuttle_analysis.git
  C02RP8FEG8WP:sfmta aleung181$ cd shuttle_analysis
  ```

* copy .csv data files to the shuttle_db directory (where the Dockerfile is). 

  > note the name of the csv files

  ```
  C02RP8FEG8WP:shuttle_analysis aleung181$ cp ~/Downloads/cnn_dim.csv shuttle_db/cnn_dim.csv
  C02RP8FEG8WP:shuttle_analysis aleung181$ cp ~/Downloads/shuttle_three_days.csv shuttle_db/shuttle_three_days.csv
  ```

* Build docker image -- this will take a couple of minutes. This downloads all the packages needed for the docker image and packages all the datafiles into the image itself -- TODO: optimize

  ```
  C02RP8FEG8WP:shuttle_analysis aleung181$ cd shuttle_db
  C02RP8FEG8WP:shuttle_db aleung181$ docker build -t shuttledb .
  <SNIP>
  Removing intermediate container 3cea1173913d
  Successfully built dc4e6307c61f
  Successfully tagged shuttledb:latest
  ```

* Run container; this will return almost immediately, but includes starting up postgres and loading all of the shuttles and cnn data, which will take around 10 minutes. During this time, postgres (e.g. psql) will not be available

  ```
  C02RP8FEG8WP:shuttle_db aleung181$ docker run -d -p 5432:5432 shuttledb
  1330b510e009fb47e3d32f4bd87b05a67e2f629958607cfdb475c079e6bf3447
  ```
  
* Verify that it's running

  ```
  C02RP8FEG8WP:shuttle_db aleung181$ docker container ls
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  1330b510e009        shuttledb           "./backup_init.sh ..."   2 minutes ago       Up 3 minutes        0.0.0.0:5432->5432/tcp   relaxed_heyrovsky
  ```
  
  > note that '1330b510e009" is the container id that can be used later to 'docker exec' commands or stop the container

  * As mentioned above, psql will not be not ready while 
    ```
    C02RP8FEG8WP:shuttle_db aleung181$ psql -U postgres -h localhost 
    psql: server closed the connection unexpectedly
      This probably means the server terminated abnormally
      before or while processing the request.
    ```
 * Once the data load has completed, use psql to perform queries
 
   ```
   C02RP8FEG8WP:shuttle_db aleung181$ psql -U postgres -h localhost 
    psql (10.1, server 9.6.6)
    Type "help" for help.

    postgres=# \c shuttle_database
    psql (10.1, server 9.6.6)
    You are now connected to database "shuttle_database" as user "postgres".
    shuttle_database=# select count(*) from shuttle_locations;
      count  
    ---------
     2640290
    (1 row)

   ```
   
 * Stop the container
   ```
   C02RP8FEG8WP:shuttle_db aleung181$ docker stop 1330b510e009
   ```
