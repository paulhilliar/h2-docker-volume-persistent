# h2-docker-volume-persistent

Example of how to use Docker volumes to:
- Run a [H2 database](https://www.h2database.com/) in server mode inside a docker container
- Have the database server store its data files on the host machine
- Connect to the database from 'outside'


H2 typically runs in embedded (mem) mode for tests etc but it can run in in TCP server mode and also persist its data files to a `dbname.db` file using a different URL format (e.g. `jdbc:h2:~/test` to host the db files in `~/test`).

### Useful commands for this example

#### Build the docker image
`docker build -t h2-docker-volume-persistent .`

#### Run the docker image

```
docker container run \
   --publish 9092:9082 \
   --volume $(pwd)/data:/usr/lib/h2 \
   --detach \
   h2-docker-volume-persistent
```

Note the volume option.  It maps the data folder `$(pwd)/data` to `/usr/lib/h2` within the container.

Notes about H2 startup options
`-baseDir /usr/lib/h2` means that all databases will be created within `/usr/lib/h2`, which is the folder we have shared as the volume using `VOLUME /usr/lib/h2`
and also mapped on container startup using `--volume $(pwd)/data:/usr/lib/h2`

`-ifNotExists` is important because without this, H2 will not allow you to create new databases remotely


### Connecting a SQL client to the H2 database
Use the command line H2 client from your host machine: `java -cp h2-*.jar org.h2.tools.Shell`

Use the URL: `jdbc:h2:tcp://localhost/./test-db` and the remaining defaults ('org.h2.Driver', 'sa', empty)

This is telling the H2 driver to connect over the TCP to localhost:9092 (which is mapped to 9092 in the running container) and the server should use a file-based database `test-db`.

The final portion of the URL (`./test-db`) tells H2 to persist the database to the default location (in h2.sh we set that to `-baseDir /usr/lib/h2`)


#### Test it out: 

sql> `create table person(id integer primary key, name varchar(100) not null);`
sql> `insert into person (id,name) values (1, 'Bob');`
sql> `select * from person;`

Then see how your data folder now contains test-db.mv.db


### Starting a shell to have a look at the running container
`docker exec -it <container-name> /bin/bash`

Try: `ls /usr/lib/h2`
Note that you can see both your H2 database file (test-db.mv.db) and also test-file.txt, which you are sharing from the host machine 

```
root@79846965307c:/# ls /usr/lib/h2
test-db.mv.db  test-file.txt
```



Note: This example was developed from the excellent example at https://github.com/nemerosa/h2
Many thanks, nemerosa :) 
