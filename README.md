# Installation guide <!-- omit in toc -->

I created this guide for the most for myself and for the size of other people who use Linux but need to use PostgreSQL with a PgAdmin graphical interface

## Summary <!-- omit in toc -->

- [asdf](#asdf)

# Podman PgAdmin4 + PostgreSQL 16

## Create a pod

```bash
NUMBER_PORT=<number port>:80 # for example: 9876:80
POD_NAME=<pod name> # for example: <pod name>

podman pod create --name <pod name> -p <number port>:80

podman pod ls # check that exist the new pod
```

## Create and run the `pgadmin` and `postgres` container inside the pod

```bash
podman run --pod=$POD_NAME -e 'PGADMIN_DEFAULT_EMAIL=<your email>' -e 'PGADMIN_DEFAULT_PASSWORD=<your password>' --name pgadmin4_v7.7 -d docker.io/dpage/pgadmin4:7.7

podman run --pod=<pod name> -v ~/.podman_volumes/<pod name>:/var/lib/postgresql/data -e 'POSTGRES_USER=<postgres username>' -e 'POSTGRES_PASSWORD=<postgres password>' --name postgresql_v16 -d docker.io/library/postgres:16.0
```

## Configure `pgadmin` to use `postgres`

Fire up your browser and head to http://localhost:<number port>.

Use the credentials you used while making the `pgadmin` container (the <your email> and <your password>).

Click on `Add Server` button.

Enter a name for your DB server and move to the next tab where you will add the host address (http://localhost) , port (the `<number port>:80`) , username (the <postgres email>)  and password (the <postgres email>).

## Useful operations

```bash
podman pod start <pod name>
podman pod stop <pod name>
```

## bash scripts

Just start and stop with these commands: `pg_psql-on`, `pg_psql-off`, need save these two functions into `.bashrc` and close and reopen the terminal window or the login session.

```bash
# Start and stop pgadmin4_v7.7-postgresql_v16 pod

function pg_psql-on()
{
    # check that pod exist
    if [ ! $( podman pod ps | grep pgadmin4_v7.7-postgresql_v16 | wc -l ) -gt 0 ]; then
        echo "ERROR: pgadmin4_v7.7-postgresql_v16 does not exist";
        return;
    fi

    output=$( podman pod ps -f name=pgadmin4_v7.7-postgresql_v16 | grep pgadmin4_v7.7-postgresql_v16 2> /dev/null | awk '{ print $3 }' )

    # check that is already running
    if [[ ${output} == "Running" ]]; then
        echo "pod pgadmin4_v7-postgresql_v16 is already Running";
        return;
    fi
    
    # check that pod is running
    if [[ ${output} == "Exited" ]]; then
        podman pod start pgadmin4_v7.7-postgresql_v16 >> /dev/null;
        echo "pod pgadmin4_v7.7-postgresql_v16 started";
    fi

    # check that is a sane pod
    if [[ ${output} == "Degraded" ]]; then
        echo "pod pgadmin4_v7.7-postgresql_v16 is Degraded";
    else
        echo "pod pgadmin4_v7-postgresql_v16 is already Running";
    fi
}

function pg_psql-off()
{
    # check that pod exist
    if [ ! $( podman pod ps | grep pgadmin4_v7.7-postgresql_v16 | wc -l ) -gt 0 ]; then
        echo "ERROR: pod named pgadmin4_v7.7-postgresql_v16 does not exist";
        return;
    fi
    
    output=$( podman pod ps -f name=pgadmin4_v7.7-postgresql_v16 | grep pgadmin4_v7.7-postgresql_v16 2> /dev/null | awk '{ print $3 }' )

    # check that pod is existed
    if [[ ${output} == "Exited" ]]; then
        echo "pod pgadmin4_v7.7-postgresql_v16 is already Exited";
    else
        podman pod stop pgadmin4_v7.7-postgresql_v16 >> /dev/null;
        echo "pod pgadmin4_v7.7-postgresql_v16 stopped";
    fi
}
```