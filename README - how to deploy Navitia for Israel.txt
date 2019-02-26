
RUNNING NAVITIA SERVER & PLAYGROUND POST INITIAL SET-UP
=======================================================
1. Run Docker on your machine

2. Open a Git bash terminal in navitia-docker-compose folder and run:
$ docker-compose-up

3. Open a Git bash terminal in navitia-playground and run:
$ npx gulp dev

4. Skip to "11. Access the Playground and start hacking!"


RUNNING NAVITIA WITH NEW GTFS FILE POST INTIAL SET-UP
=====================================================
When Navitia server is up, and you have a new GTFS file, you need to:

1. Generate a transfers.txt file - see "6. Generate a Transfers table for Navitia model"

2. Copy the new GTFS file the includes the transfers.txt file into the Navitia server container - see "9. Copy GTFS and OSM files to generate the transit graph in Navitia"

3. If needed also copy a new OSM file as well.

4. Skip to "11. Access the Playground and start hacking!"


RUNNING NAVITIA SERVER & PLAYGROUND FOR THE FIRST TIME
======================================================

1. Install Docker server on your machine: https://docs.docker.com/install/

2. Install Git with(!) the Bash terminal from https://www.atlassian.com/git/tutorials/install-git

3. Install npm: https://www.npmjs.com/get-npm

3. Get a GTFS file from  the internet. Latest version is always at israel-public-transportation.zip @ #ftp://gtfs.mot.gov.il/
   Older version can be found  at https://transitfeeds.com/p/ministry-of-transport-and-road-safety/820
   
4. Get Israel OSM (*pbf) file from https://download.geofabrik.de/asia/israel-and-palestine.html  

5. Run the docker service on your machine (Start -> Docker)

6. Generate a Transfers table for Navitia model
-----------------------------------------------
By default, when supplying Navitia with ordinary GTFS file , Navitia computes transfers only within a stop_area that includes several stops. So, if the traveller needs to walk between stop_areas (let's say Terminal 2000 bus stops on Arlozorov st. and the Savidor train station) - this would not be possible to take into consideration.

We will generate a transfrs table for the provided GTFS file.
a. Clone the following repository to your machine: git clone https://github.com/shakedk/navitia-transfers
b. Extract stops.txt from the GTFS Zip to the directory of gtfs2transfers.py
c. run "python3 gtfs2transfers.py" - takes around 50 minutes
d. Add trasnfers.txt back to the GTFS zip file

7. Add the transfers.txt file into the GTFS zip file

8. Run the Navitia Server
-------------------------
a. Clone the following repository to your machine:
$ git clone https://github.com/shakedk/navitia-docker-compose.git

b. Configure the docker container logs for rotation so they don't blow up over time:
$ cd /etc/docker
$ sudo touch daemon.json
$ vim daemon.json
Go into to edit mode by pressing I and add:
{
  "log-driver": "json-file",
  "log-opts": {"max-size": "10m", "max-file": "3"}
}
Quit with w and q
$ sudo systemctl reload docker

b. Open git bash at the repository folder and run 
$ docker-compose up
Once all the containers are up, green 'done's appear.

Sanity check: Go to http://localhost:9191/v1/coverage/default and you should see a JSON response which includes: 
status: "no_data"


9. Copy GTFS and OSM files to generate the transit graph in Navitia
-------------------------------------------------------------------
Copy the GTFS file (with the transfers file included) and the OSM files into the Navitia server containers. Run the following commands one-by-one in a NEW terminal:

$ docker cp <Files_Folder>/israel-and-palestine-latest.osm.pbf navitia-docker-compose_tyr_worker_1:/srv/ed/input/default/
$ docker cp <Files_Folder>/<GTFS-With-Transfers-file>.zip navitia-docker-compose_tyr_worker_1:/srv/ed/input/default/

The copy process takes several seconds (there's no configrmation for it).
The osm process should take about 10 minutes and ends in a message like: Task tyr.binarisation.osm2ed[86d6383d-f29f-4c24-bfad-5d8f911e8182] succeeded in 98.5151315s: None
The gtfs build takes about 30-40 minutes and ends in a message like: Task tyr.binarisation.gtfs2ed[304b2bb6-f9c9-4278-86ca-3f2c9022f6b4] succeeded in 1092.6473235s: None

9.0.1 Validate the processing of GTFS & OSM takes place - see 9.0.2 about validating the generated file exists
--------------------------------------------------------
If you don't see the messeges mentioned above in the worker logs, it probably means that tyr_beat contianer that's responsible for the worker's task managemenr is malfunctioned.

1. Try to restart tyr_beat:
a. Open Git Bash terminal and type $docker-compose up tyr_beat
b. If all goes well, tyr_beat starts and it should bring up the needed gtfs2ed and osm2ed tasks - search for them in the worker logs.
c. If not, tyr_beat is probably borken and exists shortly with exit_code = 1
c1. Stop all containers using
$docker stop $(docker ps -a -q)
c2. Delete the tyr_beat container and then the tyr_beat image:
$docker rm navitia-docker-compose_tyr_beat_1
$docker rmi docker rmi navitia/tyr-beat
c3. Re-run the docker, which will trigger a new generation of tyr_beact download: (if you need custom deployment, use the proper command)
$docker-compose up 
c4. Worker should now process the OSM & GTFS as mentioned above


9.0.2 Validate the generted graph exists
----------------------------------------
The result of the proccess should be the a graph file called default.nav.lz4 in the worker container under /srv/ed/output.
You can see it by SSHing into the container:

a. Open Git Bash terminal and type 
$docker ps
b. Copy the worker container name, e.g. "navitia-docker-compose_tyr_worker_1"
c. SSH to the container: $
docker exec -i -t navitia-docker-compose_tyr_worker_1 /bin/bash
d. Go to the folder:
$cd /srv/ed/output"
e. Check if the file there using
$ls

 In the mean time, we'll setup the Navitia playground


9.1. Copy Existing Navitia graph to another covereage
-----------------------------------------------------
The graph files of Navitia server are found in the path '/srv/ed/output' in the worker container: navitia-docker-compose_tyr_worker_1

9.1.1 Copying Between coverages on the same Navitia server
----------------------------------------------------------
Let's say you want to move the Default coverage graph to a new coverage <new-coverage>, just rename "default.nav.lz4" to "<new-coverage>.nav.lz4"

a. Copy new OSM & GTFS w/Transfers files to /srv/ed/input/default/ to generate a new default graph (see 10)
b. Restart the server (don't forget to use the custom docker-compose file - see "README - Deploy Multiple instances - Israel"


9.1.2 Copying Between coverages on the different Navitia server
----------------------------------------------------------
There's no simple option to copy files directly between containers, so you'll need to copy the files using your host machine
a. Copy the graph from the <coverage-name> to your pc:
$ docker cp navitia-docker-compose_tyr_worker_1:/srv/ed/output/<coverage>.nav.lz4 <location-on-your-machine>

b. rename to <new-coverage> name

c. Copy to the new server
$ docker cp <location-on-your-machine>/<coverage>.nav.lz4 <navitia-docker-compose_tyr_worker_1 or the name of the worker on the new server>:/srv/ed/output

d. Restart the server (don't forget to use the custom docker-compose file - see "README - Deploy Multiple instances - Israel"

10. Run the Navitia Playground
=============================
a. Clone the following repository to your machine: 
$ git clone https://github.com/shakedk/navitia-playground.git 

b. Open git bash at the repository folder and run 
$ npm install && npx bower install

Once finished, run:
npx gulp dev

11. Access the Playground and start hacking! 
===========================================
- The URL for the playground is: http://localhost:4242/play.html
- The URL for the API Server is: http://localhost:9191/v1
- Example URL for getting started with a list of all lines covered in Israel: http://localhost:4242/play.html?request=http%3A%2F%2Flocalhost%3A9191%2Fv1%2Fcoverage%2Fdefault%2Flines%3F
- Link to the Navitia API Docs: http://doc.navitia.io/


Troubleshooting:
================
1. "The region default is dead":

If the server hsan't done generating the graph yet, this is the standard error message. Wait a bit longer.

If long time passed and your're sure the data processing is done (look below for logs tutorial), stop and restart the service:
a. Stop all containers with Ctr+C or $docker stop $(docker ps -a -q)
b. Run docker-compose up

You can view the worker server logs to see if the job completled succefully:
a. Save the worker logs to a fodler on your machine by opening a terminal and running:
$ docker logs navitia-docker-compose_tyr_worker_1 >& workerLogs.txt

b. Open workerLogs.txt and search for "gtfs2ed", "osm2ed" or "ed2nav". The logs should show that these jobs are launches and succeded, e.g.:
	Line 25170: [2018-11-06 13:14:41,040] [ INFO] [    1] [        celery.worker.job] Task tyr.binarisation.gtfs2ed[37095a3c-5fbd-494a-9660-0040fb9dffb8] succeeded in 1558.0350151s: None





2. If you need to restart the deploy process from scratch and delete all data and containers run - THIS WILL REMOVE ANY DOCKER assets, NOT ONLY NATIVIA:
a. Stop all running containers by navitia: 
$ docker ps -a | grep navitia | awk '{print $1}' | xargs docker stop

b. Delete all stopped containers by Navitia: 
$ docker ps -a -f status=exited| grep navitia | awk '{print $1}' | xargs docker rm

c. Remove all unsued volumes by navitia:
$ docker volume ls | grep navitia | awk '{print $2}' | xargs docker volume rm
OR remove all volumes that are unsued by any contianer
$ docker volume prune

d. Remove all the UNUSED images (we have to move all, because the names are not specific)
docker image prune -a


3. Accessing the continaer with Bash terminal:
docker exec -i -t <container id> /bin/bash

4. copy file into AWS EC2:
$ scp -i transitanalystisrael.pem docker-israel-custom-instances.yml ec2-user@ec2-3-121-236-212.eu-central-1.compute.amazonaws.com:/home/ec2-user

