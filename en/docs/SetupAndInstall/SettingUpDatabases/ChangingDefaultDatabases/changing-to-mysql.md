# Changing to MySQL

By default, WSO2 API Manager uses the embedded H2 database as the database for storing user management and registry data. Given below are the steps you need to follow in order to use MySQL for this purpose.

!!! info
    The steps involved in installing and configuring the databases are the same irrespective of whether you are using a single node (standalone) deployment, an active-active deployment, or a distributed deployment.

## Setting up MySQL

The following sections describe how to set up a MySQL database to replace the default H2 database in your WSO2 product:

-   [Setting up the database and users](#setting-up-the-database-and-users)
-   [Setting up the drivers](#setting-up-the-drivers)
-   [Executing db scripts on MySQL database](#executing-db-scripts-on-mysql-database)

### Setting up the database and users

Follow the steps below to set up a MySQL database:

!!! note
    Note that we recommend to use Fail Over configuration over Load Balanced configuration with the MySQL clusters.

1.  Download and install [MySQL Server](http://dev.mysql.com/downloads/) in your computer.

2.  Define the hostname for configuring permissions for the new database by opening the `/etc/hosts` file and adding the following:

    !!! warning
        Do this step only if your database is not on your local machine and on a separate server.

    ``` java
    <MYSQL-DB-SERVER-IP> carbondb.mysql-wso2.com
    ```

3.  Start the MySQL service.

4.  Install mysql-client in each of the API-M servers in which WSO2 API-M is deployed. You need to do this in order to check if the servers can access the MySQL database.

    ``` java
    sudo apt install mysql-client
    mysql -h <mysqldb_host_ip> -u username -p
    ```

5.  Enter the following command in a command prompt, where `username` is the username that you will use to access the databases.

    !!! tip
        User should have the database creation privileges.

    ``` java
    mysql -u username -p
    ```

6.  When prompted, specify the password that will be used to access the databases with the username you specified.

7.  In the MySQL command prompt, create the database using the following command:

    ``` java
    mysql> create database <DATABASE_NAME>;
    ```

    !!! info
        If you are using MySQL version - 8.0.x, it is important to specify the character set as latin1 when creating databases. Failure to do this may result in an error
        It is related to the UTF-8 encoding. MySQL version 8.0.x defaults to character set UTF-8 to be friendlier to international users which uses 4 bytes per character. Hence, you must use latin1 as the character set as indicated below in the database creation commands to avoid this problem. Note that this may result in issues with non-latin characters (like Hebrew, Japanese, etc.). The database creation command should be as follows:

        ``` java
        mysql> create database <DATABASE_NAME> character set latin1;
        ```

8.  Give authorization to the user you use to access the databases as follows. For example, take `apimadmin` as the user.

    ``` java
    GRANT ALL ON regdb.* TO apimadmin@localhost IDENTIFIED BY "apimadmin";
    ```

    !!! info
        If you are using MySQL version - 8.0.x, use following commands to create the user and grant authorization:

        ``` java
        CREATE USER 'apimadmin'@'localhost' IDENTIFIED BY 'apimadmin';
        ```

        ``` java
        GRANT ALL ON APIM.* TO 'apimadmin'@'localhost';
        ```

9.  Once you have finalized the permissions, reload all the privileges by executing the following command:

    ``` java
    FLUSH PRIVILEGES;
    ```

10.  Log out from the MySQL command prompt by executing the following command:

    ``` java
    quit;
    ```

### Setting up the drivers

1. Unzip the WSO2 API Manager pack. Let's call it `<API-M_HOME>`.

2. Download the MySQL Java connector [JAR file](https://dev.mysql.com/downloads/connector/j/), and extract it.

3. Copy it to the `<API-M_HOME>/repository/components/lib/` directory in all the nodes of the cluster.

!!! tip
    Be sure to use the connector version that is supported by the MySQL version you use. If you come across any issues due to version incompatibility, follow the steps below:

    1.  Shut down the server and remove all existing connectors from `<API-M_HOME>/repository/components/lib` and `<API-M_HOME>/repository/components/dropins`.
    2.  Download the connector JAR that is compatible with your current MySQL version.
    3.  Copy the JAR file **only to** `<API-M_HOME>/repository/components/lib` location. Files will be copied automatically to the dropins folder during the server startup.

### Executing db scripts on MySQL database

To run the database script against the database you created, login to the MySQL client and point to the corresponding database.

``` java
use shareddb;
```

Execute the mysql.sql database script against the pointed database using following command.

``` java
mysql> source <path to the script>\mysql.sql;
```
!!! info
    About using MySQL in different operating systems

    For users of Microsoft Windows, when creating the database in MySQL, it is important to specify the character set as latin1. Failure to do this may result in an error (error code: 1709) when starting your cluster. This error occurs in certain versions of MySQL (5.6.x) and is related to the UTF-8 encoding. MySQL originally used the latin1 character set by default, which stored characters in a 2-byte sequence. However, in recent versions, MySQL defaults to UTF-8 to be friendlier to international users. Hence, you must use latin1 as the character set as indicated below in the database creation commands to avoid this problem. Note that this may result in issues with non-latin characters (like Hebrew, Japanese, etc.). The following is how your database creation command should look.

    ``` java
    mysql> create database <DATABASE_NAME> character set latin1;
    ```

    For users of other operating systems, the standard database creation commands are sufficient. For these operating systems, the following is how your database creation command should look.

    ``` java
    mysql> create database <DATABASE_NAME>;
    ```

!!! note
    If you are using MySQL to configure your datasources, we recommend that you use a case sensitive database collation. For more information, see the [MySQL Official Manual](https://dev.mysql.com/doc/refman/5.7/en/charset-mysql.html) . The default database collation, which is `latin1_swedish_ci` , is case insensitive. However, you need to maintain case sensivity for database collation, because when the database or table has a case-insensitive collation in MySQL 5.6 or 5.7, if a user creates an API with letters using mixed case, deletes the API, and then creates another API with the same name, but in lower case letters, then the later created API loses its permission information, because when deleting the API, it keeps the Registry collection left behind.

    This issue could be avoided if you use a case sensitive collation for database and tables. In that case, when creating the second API (which has the same name, but is entirely in lowercase letters), it will create a new record with the lowercase name in the `UM_PERMISSION` table.

!!! note
    Additional notes

    -   Ensure that MySQL is configured so that all nodes can connect to it.
    -   To access the databases from remote instances, its required to grant permission to the relevant username defined in the `<API-M_HOME>/repository/conf/deployment.toml` file under `[database.shared_db]` or `[database.apim_db]` elements, by using the grant command. See the following sample commands.

```tab="Format"
mysql> create database apimgtdb;
mysql> use apimgtdb;
mysql> source <API-M_HOME>/dbscripts/apimgt/mysql.sql;
mysql> grant all on apimgtdb.* TO '<username>'@'%' identified by '<password>';
    
mysql> create database shareddb;
mysql> use shareddb;
mysql> source <API-M_HOME>/dbscripts/mysql.sql;
mysql> grant all on shareddb.* TO '<username>'@'%' identified by '<password>';
     
mysql> create database mbstoredb;
mysql> use mbstoredb;
mysql> source <API-M_HOME>/dbscripts/mb-store/mysql-mb.sql;
mysql> grant all on mbstoredb.* TO '<username>'@'%' identified by '<password>';
```

``` tab="Example"
mysql> create database apimgtdb;
mysql> use apimgtdb;
mysql> source <API-M_HOME>/dbscripts/apimgt/mysql.sql;
mysql> grant all on apimgtdb.* TO 'wso2user'@'%' identified by 'wso2123';

mysql> create database shareddb;
mysql> use shareddb;
mysql> source <API-M_HOME>/dbscripts/mysql.sql;
mysql> grant all on shareddb.* TO 'wso2user'@'%' identified by 'wso2123';

mysql> create database mbstoredb;
mysql> use mbstoredb;
mysql> source <API-M_HOME>/dbscripts/mb-store/mysql-mb.sql;
mysql> grant all on mbstoredb.* TO 'wso2user'@'%' identified by 'wso2123';
```

!!! note
    In the sample commands above, its assumed that the username and password defined in the datasource configurations in `<API-M_HOME>/repository/conf/deployment.toml` file is **wso2user** and **wso2123** respectively.

## Changing to MySQL

-   [Creating the datasource connection to MySQL](#creating-the-datasource-connection-to-mysql)
-   [Creating database tables](#creating-database-tables)

### Creating the datasource connection to MySQL

A datasource is used to establish the connection to a database. By default, `WSO2_SHARED_DB` and `WSO2AM_DB` datasources are configured in the `deployment.toml` file for the purpose of connecting to the default H2 databases.

After setting up the MySQL database to replace the default H2 database, either change the default configurations of the `WSO2_SHARED_DB` and `WSO2AM_DB` datasources, or configure a new datasource to point it to the new database as explained below.

!!! note
    **If you are configuring API-M in a distributed setup** , do the changes in all the WSO2 API-M components.

Follow the steps below to change the type of the default datasources.

1.  Open the `<API-M_HOME>/repository/conf/deployment.toml` configuration file and locate the `[database.shared_db]` and `[database.apim_db]` configuration elements.

2.  You simply have to update the URL pointing to your MySQL database, the username, and password required to access the database and the MySQL driver details as shown below.

    | Element                       | Description                                                 |
    |-------------------------------|-------------------------------------------------------------|
    | **type**                      | The database type used                                      |
    | **url**                       | The URL of the database. The default port for MySQL is 3306 |
    | **username** and **password** | The name and password of the database user                  |
    | **driverClassName**           | The class name of the database driver                       |

    Sample configuration is shown below:

    ``` tab="Format"
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/<DATABASE_NAME>"
    username = "regadmin"
    password = "regadmin"
    ```

    ``` tab="Example"
    [database.shared_db]
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/shared_db"
    username = "regadmin"
    password = "regadmin"

    [database.apim_db]
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/apim_db"
    username = "apimadmin"
    password = "apimadmin"
    ```

    !!! info
        If you are using MySQL version - 8.0.x, you should add the driver name in the configuration as:
        ``` java
        driver="com.mysql.cj.jdbc.Driver"
        ```

    After adding these configurations, you could see that master-datasource.xml which located in <API-M_HOME>/repository/conf/datasource will be updated as follows after server startup:

    ``` xml
    <datasource>
        <name>WSO2_SHARED_DB</name>
        <description>The datasource used for registry and user manager</description>
        <jndiConfig>
            <name>jdbc/SHARED_DB</name>
        </jndiConfig>
        <definition type="RDBMS">
            <configuration>
                <url>jdbc:mysql://localhost:3306/<DATABASE_NAME></url>
                <username>regadmin</username>
                <password>regadmin</password>
                <driverClassName>com.mysql.cj.jdbc.Driver</driverClassName>
                <testOnBorrow>true</testOnBorrow>
                <maxWait>60000</maxWait>
                <defaultAutoCommit>true</defaultAutoCommit>
                <validationInterval>30000</validationInterval>
                <maxActive>50</maxActive>
            </configuration>
        </definition>
    </datasource>

    <datasource>
        <name>WSO2AM_DB</name>
        <description>The datasource used for API Manager database</description>
        <jndiConfig>
            <name>jdbc/WSO2AM_DB</name>
        </jndiConfig>
        <definition type="RDBMS">
            <configuration>
                <url>jdbc:mysql://localhost:3306/<DATABASE_NAME></url>
                <username>apimadmin</username>
                <password>apimadmin</password>
                <driverClassName>com.mysql.cj.jdbc.Driver</driverClassName>
                <testOnBorrow>true</testOnBorrow>
                <maxWait>60000</maxWait>
                <defaultAutoCommit>true</defaultAutoCommit>
                <validationInterval>30000</validationInterval>
                <maxActive>50</maxActive>
            </configuration>
            </definition>
    </datasource>       
    ```

3.  You can update the configuration elements given below for your database connection.

    | Element                | Description                                                                                                                                                                                                                                                                                                                                  |
    |------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | **maxActive**          | The maximum number of active connections that can be allocated at the same time from this pool. Enter any negative value to denote an unlimited number of active connections.                                                                                                                                                                |
    | **maxWait**            | The maximum number of milliseconds that the pool will wait (when there are no available connections) for a connection to be returned before throwing an exception. You can enter zero or a negative value to wait indefinitely.                                                                                                              |
    | **minIdle**            | The minimum number of active connections that can remain idle in the pool without extra ones being created, or enter zero to create none.                                                                                                                                                                                                    |
    | **testOnBorrow**       | The indication of whether objects will be validated before being borrowed from the pool. If the object fails to validate, it will be dropped from the pool, and another attempt will be made to borrow another.                                                                                                                              |
    | **validationQuery**    | The SQL query that will be used to validate connections from this pool before returning them to the caller.                                                                                                                                                                                                                                  |
    | **validationInterval** | The indication to avoid excess validation, and only run validation at the most, at this frequency (time in milliseconds). If a connection is due for validation but has been validated previously within this interval, it will not be validated again.                                                                                      |
    | **defaultAutoCommit**  | This property is **not** applicable to the Carbon database in WSO2 products because auto committing is usually handled at the code level, i.e., the default auto commit configuration specified for the RDBMS driver will be effective instead of this property element. Typically, auto committing is enabled for RDBMS drivers by default. When auto committing is enabled, each SQL statement will be committed to the database as an individual transaction, as opposed to committing multiple statements as a single transaction.|                                                              
    | **commitOnReturn**     | If `defaultAutoCommit =false`, then you can set `commitOnReturn =true`, so that the pool can complete the transaction by calling the commit on the connection as it is returned to the pool. However, If `rollbackOnReturn =true` then this attribute is ignored. The default value is false.|
    | **rollbackOnReturn** | If `defaultAutoCommit =false`, then you can set `rollbackOnReturn =true` so that the pool can terminate the transaction by calling rollback on the connection as it is returned to the pool. The default value is false.|

    !!! info
        For more information on other parameters that can be defined in the `<API-M_HOME>/repository/conf/datasources/master-datasources.xml` file, see [Tomcat JDBC Connection Pool](http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Tomcat_JDBC_Enhanced_Attributes).

### Creating database tables

To create the database tables, connect to the database that you created earlier and run the relevant scripts.

1.  To create tables in the registry and user manager database ( `WSO2_SHARED_DB` ), execute the relevant script as shown below.

    ``` powershell
    mysql -u regadmin -p -Dshared_db < '<API-M_HOME>/dbscripts/mysql.sql';
    ```

2.  To create tables in the apim database ( `WSO2AM_DB` ), execute the relevant script as shown below.

    ``` powershell
    mysql -u apimadmin -p -Dapim_db < '<API-M_HOME>/dbscripts/apimgt/mysql.sql';
    ```

    !!! note
        `<API-M_HOME>/dbscripts/mb-store/mysql.sql` is the script that should be used when creating the tables in `WSO2_MB_STORE_DB` database.

3.  Restart the server.

!!! note
    To give the Key Manager, Publisher, and Developer Portal components access to the user management data with shared permissions, JDBCUserStoreManager has been configured by default. For more information, refer [Configuring Userstores](../../../Administer/ProductConfigurations/ConfiguringUserStores/configuring-user-stores.md).