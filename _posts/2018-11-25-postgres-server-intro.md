---
title: PostgreSQL spatial database set-up and remote server access
date: November 25, 2018
layout: single
permalink: /intro-to-spatial-databases/
---

Setting up your first database can be daunting - however it is likely one of the most useful and rewarding things you can do for your project. In this tutorial I will cover the basics of setting up a spatial database in PostgreSQL (PostGIS), and will walk you through how to do that on a server which runs on your local network. This way the database can be accessed by all members of that local network (e.g. your organisation). 

## Server setup

### Basics

1. Log into your server and download the latest version [PostgreSQL](https://www.postgresql.org/). After that is done open PgAdmin 4 and create a database - set the host of the database to `localhost`.  
<br>
2. If you are interested in also storing spatial data in this database (e.g. geometry, coordinates, rasters) then you must initialise the `PostGIS` extension in the database by executing the following command in the PgAdmin 4 SQL editor `create extension postgis`. <br><br>**Tip:** *To see all currently installed extensions open your database schema and go to extensions.*
<br>
3. Now, we must find out the IP address of our server (unless you already know it), so that we will be able to connect to this database from another computer. To find the IP address, open a command prompt or terminal and execute the following commands based what operating system is installed on your server: <br>
**Windows:** `ipconfig`
**Mac/Linux:** `ifconfig` <br>

**Note:** *Write the IP address down as we will need it in the next step. It should look something like this `172.14.1.16`*

### Access configuration

Before other computers can access your database, PostgreSQL needs to be configured to listen for connections from other IP addresses. To do this go to your PostgreSQL installation directory and open `postgresql.config` in a text-editor - once inside the file, set `listen_addresses = '*'`.

Next, we need to define what specific IP addresses are allowed to connect and what security protocol those connections should follow. For this edit the [`pg_hba.config`](https://www.postgresql.org/docs/devel/auth-pg-hba-conf.html) file also located inside your PostgreSQL installation directory. Here add the following to the end of the document to allow connections from all IPs:

```bash
host    all             all             0.0.0.0/0               md5
host    all             all             ::/0                    md5
```

To allow connections from specific IP addresses only, add them like below (recommended unless your server is on a private network):

```bash
host    all             all             172.16.1.138/32         md5
host    all             all             ::1/128                 md5
```

**note:** *`md5` asks for username and password verification, whereas setting to `trust` requires no credentials. These credentials are the same as the ones you can create in your database - for more on information, consult this tutorial on [Role Management in PostgreSQL](http://www.postgresqltutorial.com/postgresql-roles/)*

In order for the above changes to take effect, apart from saving the edited files you now also need to restart the PostgreSQL server. Various methods to achieve that are shown [here.](https://www.pokertracker.com/guides/PT4/troubleshooting/restart-the-postgresql-server)

You should now be good to go!

## Establishing a connection

### PgAdmin 4

1. Launch PgAdmin 4 and create a new server.
2. Give this server a name and in `host name/address` specify the IP address of the target machine. 
3. Set the `port` to default 5432.
4. Enter username and password.

### PSQL

If using the `psql` shell that comes with the PostgreSQL installation when prompted enter the same information as above.

If using the `psql` terminal utility you can access your database through the following command:

```bash
psql -U <enter username> -h <enter IP address of server> -d <enter database name>
```

### QGIS

1. Launch QGIS and create a new postGIS connection in the QGIS browser.
2. Give the connection a `name` of your choice.
3. For `host` specify the IP address of the server.
4. Set the `port` to default 5432.
5. Type the name of the database you wish to connect to.
6. Open basic authentication options and enter the username and password for the database.
7. Test your connection.
8. Once successful, press OK and all data is now available to you in the QGIS browser.

### Disabling the firewall

Finally if you are still having some issues connecting to your database it may be that there is a firewall installed on your server that is blocking incoming connections to port 5432. In order to fix this, you must thus open port 5432 and allow incoming connections from other machines. 



**Opening port 5432 on Windows:**

Go to: `Control Panel / Windows Defender Firewall / Incoming Connections / Create New Rule`

**Then**

1. Give the rule a name
2. Allow all incoming connections
3. Use `TCP` protocol
4. Specify port 5432
5. Save rule

For other operating systems the process will be very similar to the one above. For more information on how to open ports on Mac OSX see this [article](https://www.macworld.co.uk/how-to/mac-software/how-open-specific-ports-in-os-x-1010-firewall-3616405/).

## Uploading data

Now that you've connected to the database there are various ways to upload your data.

### QGIS

1. Launch QGIS and upload data to the browser.
2. To upload data to server open database manager.
3. Select a server under the `postGIS` menu and click on `import layer/file`.
4. Select your data, enter table name (and other options if desired) and click ok.

### PSQL

Vector and raster data can also be uploaded via the `psql` shell.

#### `shp2pgsql` for vector data

To create a new table using a shapefile, execute the following command (alternatively if you wish to append to an existing table add the `-a` option).

```bash
# upload a shapefile
shp2pgsql -a -s <enter srid> -I <enter full file path> <enter desired table name> | psql -h <enter host> -U <enter username> -p <enter port> -d <enter database name>
```

The above command generates an SQL script that inserts our shapefile's data into our selected table. This SQL script is then piped straight into the `psql` utility where we specify the username and database. This completes the uploading of our data into the selected table within the PostgreSQL database.

#### `raster2pgsql` for raster data

To create a new table using a raster, execute the following command (alternatively if you wish to append to an existing table add the `-a` option).

```bash
# upload a raster
raster2pgsql -s <enter srid> -d -C -I -M -Y -l 2,4,8,16,32,64,128,256 <enter full file path> -t auto <enter desired table name> | psql -h <enter host> -U <enter username> -p <enter port> -d <enter database name>
```

**note:** This example exhibits various additional options which are used to create [raster pyramids](http://edndoc.esri.com/arcsde/9.2/concepts/rasters/basicprinciples/pyramids.htm) which is used to increase the upload speed of large rasters. For more details see this [article.](https://duncanjg.wordpress.com/2012/11/20/the-basics-of-postgis-raster/)

## Backing up your data

Finally you can create a backup of your entire database easly from within PgAdmin 4. Here right-clicked your database and select the `backup` option. This will generate an SQL script containing the entire database. To then upload the database on another server simply enter pgAdmin 4, create a new server, right-click and select restore. Now select the SQL file from earlier and the database will be regenerated in this location.