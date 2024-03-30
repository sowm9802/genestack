# MariaDB Operations

Tips and tricks for managing and operating the MariaDB cluster within a Genestack environment.

## Connect to the database

Sometimes an operator may need to connect to the database to troubleshoot things or otherwise make modifications to the databases in place. The following command can be used to connect to the database from a node within the cluster.

``` shell
mysql -h $(kubectl -n openstack get service maxscale-galera -o jsonpath='{.spec.clusterIP}') \
      -p$(kubectl --namespace openstack get secret maxscale -o jsonpath='{.data.password}' | base64 -d) \
      -u maxscale-galera-client
```

!!! info

    The following command will leverage your kube configuration and dynamically source the needed information to connect to the MySQL cluster. You will need to ensure you have installed the mysql client tools on the system you're attempting to connect from.

## Dumping the databases

When running `mysqldump` or `mariadbdump` the following commands can be useful for generating a quick backup.

``` shell
mysqldump --host=$(kubectl -n openstack get service mariadb-galera -o jsonpath='{.spec.clusterIP}')\
          --user=root \
          --password=$(kubectl --namespace openstack get secret mariadb -o jsonpath='{.data.root-password}' | base64 -d) \
          --single-transaction \
          --routines \
          --triggers \
          --events \
          ${DATABASE_NAME} \
          --result-file=/tmp/${DATABASE_NAME}-$(date +%s).sql
```

!!! example "Dump all databases as individual files in `/tmp`"

    ``` shell
    mysql -h $(kubectl -n openstack get service mariadb-galera -o jsonpath='{.spec.clusterIP}') \
          -u root \
          -p$(kubectl --namespace openstack get secret mariadb -o jsonpath='{.data.root-password}' | base64 -d) \
          -e 'show databases;' \
          --column-names=false \
          --vertical | \
              awk '/[:alnum:]/' | \
                  xargs -i mysqldump --host=$(kubectl -n openstack get service mariadb-galera -o jsonpath='{.spec.clusterIP}') \
                  --user=root \
                  --password=$(kubectl --namespace openstack get secret mariadb -o jsonpath='{.data.root-password}' | base64 -d) \
                  --single-transaction \
                  --routines \
                  --triggers \
                  --events \
                  {} \
                  --result-file=/tmp/{}-$(date +%s).sql
    ```

!!! example "Restoring a database"

    ``` shell
    mysql -h $(kubectl -n openstack get service mariadb-galera -o jsonpath='{.spec.clusterIP}') \
        -u root \
        -p$(kubectl --namespace openstack get secret mariadb -o jsonpath='{.data.root-password}' | base64 -d) \
        ${DATABASE_NAME} < /tmp/${DATABASE_FILE}
    ```