# BlueSky Web

The blueskyweb package contains a tornado web service, wrapping
[bluesky][https://github.com/pnwairfire/bluesky],
that can be started by simply running ```bsp-web```.






## External Dependencies

### mongodb

BlueSky web connects to a mongodb database to query met data availability
and to enqueue background plumerise and dispersion runs.
You just need to provide the url of one that is running.

### Docker

Docker is required to run the bluesky web system - mongodb,
api web service, output web service, bsp workers, and ofelia.





## Development

### Setup and Run in Docker

    git clone git@github.com:pnwairfire/bluesky-web.git pnwairfire-bluesky-web
    cd pnwairfire-bluesky-web
    pip install -r requirements-dev.txt
    ./reboot --rebuild

When using docker, arlindex will automatically be updated every
15 minutes using the mcuadros/ofelia docker image.
If you don't want to wait for it to run, manually run it with:

    docker exec bluesky-web-worker \
        arlindexer -d DRI4km -r /data/Met/CANSAC/4km/ARL/ \
        -m mongodb://blueskyweb:blueskywebmongopassword@mongo/blueskyweb \
        --mongo-ssl-ca-certs /etc/ssl/bluesky-web-client.pem
    docker exec bluesky-web-worker \
        arlindexer -d NAM84 -r /data/Met/NAM/12km/ARL/ \
        -p NAM84_ARL_index.csv \
        -m mongodb://blueskyweb:blueskywebmongopassword@mongo/blueskyweb \
        --mongo-ssl-ca-certs /etc/ssl/bluesky-web-client.pem

Check that service is running:

    curl http://localhost:8887/bluesky-web/api/ping

### Tail logs

    cd /path/to/airfire-bluesky-web
    find ./dev/logs/ -name *.log -exec tail -f "$file" {} +

### dockerized ipython session

    docker run --rm -ti -v $PWD:/usr/src/blueskyweb/ bluesky-web ipython



## Tests

### Unit Tests

    docker run --rm -ti -v $PWD:/usr/src/blueskyweb/ bluesky-web py.test test

## Ad Hoc tests

These can be run outside of docker. See the helpstrings for
the following two scripts for examples

    ./dev/scripts/web-regression-test.sh
    ./dev/scripts/test-async-request.py -h





## Fabric

To see list tasks:

    fab -l

To see documentation for a specific task, use the '-d' option. E.g.:

    fab -d deploy

### Test Environment

First time:

    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A deploy
    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A start
    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A configure_web_apache_proxy
    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A configure_output_web_apache_proxy

Subsequent deployments:

    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A deploy

(Note that the deploy tasks takes care of restarting the docker containers.)

Check status

    FABRIC_USER=$USER BLUESKY_WEB_ENV=test fab -A check_status

### Production Environment

Same as test env, but substitude 'production' for 'test' in each
command.

### All envs

    ./deploy-all-envs




## Manually checking deployment

Check processes and look at files installed on web service server,
e.g. haze

    ssh server1 docker ps -a |grep bluesky-web-test
    ssh server1 cat /etc/docker-compose/bluesky-web-test/docker-compose.yml
    ssh server1 cat /etc/bluesky-web/test/config.json

And on worker servers, e.g. judy

    ssh server2 docker ps -a |grep bluesky-web-test
    ssh server2 cat /etc/docker-compose/bluesky-web-test/docker-compose.yml
    ssh server2 cat /etc/bluesky-web/test/config.json
    ssh server2 cat /etc/bluesky-web/test/ofelia/config.ini




## APIs

See [APIs](doc/API.md)




## Manually connecting to mongodb

start session

    docker exec -ti bluesky-web-mongo \
        mongo -u blueskyweb -p blueskywebmongopassword --ssl \
        --sslCAFile /etc/ssl/bluesky-web-mongod.pem \
        --sslAllowInvalidHostnames --sslAllowInvalidCertificates \
        blueskyweb

run query on command line

    docker exec -ti bluesky-web-mongo \
        mongo -u blueskyweb -p blueskywebmongopassword --ssl \
        --sslCAFile /etc/ssl/bluesky-web-mongod.pem \
        --sslAllowInvalidHostnames --sslAllowInvalidCertificates \
        blueskyweb --eval 'db.met_files.findOne()'




## Checking on Production System

    docker run --rm -ti bluesky-web ./bin/bsp-web-manage-runs-db \
        -m mongodb://blueskyweb:blueskywebmongopassword@db.blueskywebhost.com:27886/ \
        -a get-runs  --status enqueud

or just load api

    https://www.blueskywebhost.com/bluesky-web/api/v4.1/runs/enqueued/
