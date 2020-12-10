# mysql-extend-image
Demo scripts to extend Red Hat MYSQL runtime image

Following is an extract from the original README.md:
https://github.com/sclorg/mysql-container/blob/973fb0e587de6fad7c461558889e67ac2c9789dc/5.7/root/usr/share/container-scripts/mysql/README.md

Extending image
---------------
This image can be extended in Openshift using the `Source` build strategy or via the standalone
[source-to-image](https://github.com/openshift/source-to-image) application (where available).
For this, we will assume that you are using the `rhscl/mysql-57-rhel7` image,
available via `mysql:5.7` imagestream tag in Openshift.

For example, to build a customized MySQL database image `my-mysql-rhel7`
with a configuration from `https://github.com/sclorg/mysql-container/tree/master/examples/extend-image` run:

```
$ oc new-app mysql:5.7~https://github.com/sclorg/mysql-container.git \
        --name my-mysql-rhel7 \
        --context-dir=examples/extend-image \
        --env MYSQL_OPERATIONS_USER=opuser \
        --env MYSQL_OPERATIONS_PASSWORD=oppass \
        --env MYSQL_DATABASE=opdb \
        --env MYSQL_USER=user \
        --env MYSQL_PASSWORD=pass
```

or via s2i:

```
$ s2i build --context-dir=examples/extend-image https://github.com/sclorg/mysql-container.git rhscl/mysql-57-rhel7 my-mysql-rhel7
```

The directory passed to Openshift can contain these directories:

`mysql-cfg/`
    When starting the container, files from this directory will be used as
    a configuration for the `mysqld` daemon.
    `envsubst` command is run on this file to still allow customization of
    the image using environmental variables

`mysql-pre-init/`
    Shell scripts (`*.sh`) available in this directory are sourced before
    `mysqld` daemon is started.

`mysql-init/`
    Shell scripts (`*.sh`) available in this directory are sourced when
    `mysqld` daemon is started locally. In this phase, use `${mysql_flags}`
    to connect to the locally running daemon, for example `mysql $mysql_flags < dump.sql`

Variables that can be used in the scripts provided to s2i:

`$mysql_flags`
    arguments for the `mysql` tool that will connect to the locally running `mysqld` during initialization

`$MYSQL_RUNNING_AS_MASTER`
    variable defined when the container is run with `run-mysqld-master` command

`$MYSQL_RUNNING_AS_SLAVE`
    variable defined when the container is run with `run-mysqld-slave` command

`$MYSQL_DATADIR_FIRST_INIT`
    variable defined when the container was initialized from the empty data dir

During `s2i build` all provided files are copied into `/opt/app-root/src`
directory into the resulting image. If some configuration files are present
in the destination directory, files with the same name are overwritten.
Also only one file with the same name can be used for customization and user
provided files are preferred over default files in
`/usr/share/container-scripts/mysql/`- so it is possible to overwrite them.

Same configuration directory structure can be used to customize the image
every time the image is started using `podman run`. The directory has to be
mounted into `/opt/app-root/src/` in the image
(`-v ./image-configuration/:/opt/app-root/src/`).
This overwrites customization built into the image.


