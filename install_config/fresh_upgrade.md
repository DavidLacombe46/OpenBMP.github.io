---
title: Fresh Install Upgrade
---

# Fresh Install to Upgrade

> You can separate the compose file to be deployed using many VMs.  In order to do that, you'll need to
adjust the FQDNs and ports that are used to access resources, such as Kafka and Postgres.

1. Define ENV

    ```warning
   You MUST export this var for the below commands to work.
    ```
   
    ```tip
   Run all the below commands as **root** instead of using sudo. You can use sudo, but you'll need
   to specify files for any wildcard. 
   ```

   ```
   export OBMP_DATA_ROOT=/var/openbmp
   ```

2. Shutdown and remove current deployment: 

   ```
   OMP_DATA_ROOT=${OBMP_DATA_ROOT}  docker-compose -p obmp down
   ```

3. Get latest docker compose

   ```
   wget --backups=3 https://raw.githubusercontent.com/OpenBMP/obmp-docker/main/docker-compose.yml
   ```

4. Update the docker-compose.yml file variables and volumes based on your previous compose file.
   You can use ```diff``` to see the differences that need to be merged/updated.

    ```
   diff -u docker-compose.yml.1 docker-compose.yml
   ```

5. Allow DB to be reinitialized:

   ```
   rm -f ${OBMP_DATA_ROOT}/config/do_not_init_db
   ```

7. Remove current postgres data. **Make sure the files are deleted.**. If using ```sudo```, the wildcard
   doesn't work.  You will need to use ```sudo bash -c ...``` instead. 

   ```
   rm -rf ${OBMP_DATA_ROOT}/postgres/data/*
   rm -rf ${OBMP_DATA_ROOT}/postgres/ts/*
   ```
   
8. Remove Kafka/Zookeeper data:

   ```
   rm -rf ${OBMP_DATA_ROOT}/kafka-data/*
   rm -rf ${OBMP_DATA_ROOT}/zk-data/*
   rm -rf ${OBMP_DATA_ROOT}/zk-log/*
   ```

9. Update Grafana.
   ```
   rm -rf ${OBMP_DATA_ROOT}/grafana/plugins ${OBMP_DATA_ROOT}/grafana/alerting ${OBMP_DATA_ROOT}/grafana/grafana.db
   rm -rf ${OBMP_DATA_ROOT}/grafana/dashboards/
   rm -rf ${OBMP_DATA_ROOT}/grafana/provisioning/
   
   git clone https://github.com/OpenBMP/obmp-grafana.git
   # or do: git pull
  
   cp -r obmp-grafana/dashboards obmp-grafana/provisioning ${OBMP_DATA_ROOT}/grafana/
   chmod go+xr -R ${OBMP_DATA_ROOT}/grafana/
   ```

   ```danger
    Make sure the files copied are owned by the container user. If not, provisioning will not load.
    ```
   
   
11. Start the new/upgraded version of OBMP. This will reinitialize the DB.  It does take a little time
    on initial start.

    ```
    OBMP_DATA_ROOT=${OBMP_DATA_ROOT} docker-compose -p obmp up -d
    ```  