# docker-irods-resource
Docker implementation of iRODS Resource Server

## Supported tags and respective Dockerfile links

- 4.1.9, latest ([4.1.9/Dockerfile](https://github.com/mjstealey/docker-irods-resource/blob/master/4.1.9/Dockerfile))
- 4.1.8 ([4.1.8/Dockerfile](https://github.com/mjstealey/docker-irods-resource/blob/master/4.1.8/Dockerfile))
- 4.1.7 ([4.1.7/Dockerfile](https://github.com/mjstealey/docker-irods-resource/blob/master/4.1.7/Dockerfile))

### Pull image from dockerhub

```
docker pull mjstealey/docker-irods-resource:latest
```

### Usage:

**Example 1.** iRODS resource servers assume that there is an already running instance of an iCAT server reachable over the network.

In this example we've previously launched a daemonized instance of [docker-irods-icat:4.1.8](https://github.com/mjstealey/docker-irods-icat) and have specified that both it's docker name and hostname are **icat**:
```
$ docker run -d --name icat \
  --hostname icat \
  mjstealey/docker-irods-icat:4.1.8
```

When launching our resource server we want to match the version of the iRODS iCAT server, in this example v.4.1.8, and provide a few docker attributes to allow the conainer to bind with the already running iCAT instance. We'll use the **--link** attribute to specify which container it should have IP information for, the **--hostname** attribute to provide a clean name to the created resource, as well as specify the environment variable **IRODS_ICAT_HOST_NAME** to match the hostname we gave to the docker-irods-icat:4.1.8 instance.
```
$ docker run --name resource \
  --hostname resource \
  -e IRODS_ICAT_HOST_NAME=icat \
  --link icat:icat \
  mjstealey/docker-irods-resource:4.1.8
```
This call can also be daemonized with the **-d** flag, which would most likely be used in an actual environment.

On completion a running container named **resource** is spawned with the following configuration:
```
-------------------------------------------
iRODS Port:                 1247
Range (Begin):              20000
Range (End):                20199
Vault Directory:            /var/lib/irods/iRODS/Vault
zone_key:                   TEMPORARY_zone_key
negotiation_key:            TEMPORARY_32byte_negotiation_key
Control Plane Port:         1248
Control Plane Key:          TEMPORARY__32byte_ctrl_plane_key
Schema Validation Base URI: https://schemas.irods.org/configuration
Administrator Username:     rods
-------------------------------------------
-------------------------------------------
iCAT Host:    icat
iCAT Zone:    tempZone
-------------------------------------------
```

Use the **docker exec** call to at the terminal interact with the container. Add the user definition of **-u irods** to specify that commands should be run as the **irods** user.

- Sample **iadmin lr**:
  ```
  $ docker exec -u irods icat iadmin lr
  bundleResc
  demoResc
  resourceResource
  $ docker exec -u irods resource iadmin lr
  bundleResc
  demoResc
  resourceResource
  ```
From this call you can see the newly launched **resourceResource** from both the **icat** server as well as the **resource** server.

- Sample **iadmin lr resourceResource**
  ```
  $ docker exec -u irods icat iadmin lr resourceResource
  resc_id: 10001
  resc_name: resourceResource
  zone_name: tempZone
  resc_type_name: unixfilesystem
  resc_net: resource
  resc_def_path: /var/lib/irods/iRODS/Vault
  free_space:
  free_space_ts Never
  resc_info:
  r_comment:
  resc_status:
  create_ts 2016-10-03.16:44:54
  modify_ts 2016-10-03.16:44:55
  resc_children:
  resc_context:
  resc_parent:
  resc_objcount: 0
  ```
  
**Example 2.** Use an environment file to pass the required environment variables for the iRODS `setup_irods.sh` call.
```
$ docker run --name resource \
  --env-file sample-env-file.env \
  --hostname resource \
  --link icat:icat \
  mjstealey/docker-irods-resource:4.1.8
```
- Using sample environment file named `sample-env-file.env` (Update as required for your iRODS installation)

  ```bash
  IRODS_SERVICE_ACCOUNT_NAME=irods
  IRODS_SERVICE_ACCOUNT_GROUP=irods
  IRODS_PORT=1247
  IRODS_PORT_RANGE_BEGIN=20000
  IRODS_PORT_RANGE_END=20199
  IRODS_VAULT_DIRECTORY=/var/lib/irods/iRODS/Vault
  IRODS_SERVER_ZONE_KEY=TEMPORARY_zone_key
  IRODS_SERVER_NEGOTIATION_KEY=TEMPORARY_32byte_negotiation_key
  IRODS_CONTROL_PLANE_PORT=1248
  IRODS_CONTROL_PLANE_KEY=TEMPORARY__32byte_ctrl_plane_key
  IRODS_SCHEMA_VALIDATION=https://schemas.irods.org/configuration
  IRODS_ICAT_SERVER_ADMINISTRATOR_USER_NAME=rods
  IRODS_ICAT_HOST_NAME=icat
  IRODS_ICAT_ZONE_NAME=tempZone
  IRODS_ICAT_SERVER_ADMINISTRATOR_PASSWORD=rods
  ```
  
This call can also be daemonized with the **-d** flag, which would most likely be used in an actual environment.
The outcome of this call would be identical to that described in Example 1, with the same results for the `docker exec -u irods icat iadmin` calls.
