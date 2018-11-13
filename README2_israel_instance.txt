

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
When NAvitia server is up, and you have a new GTFS file, you need to:

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
a. Clone the following repository to your machine: git clone https://github.com/CanalTP/transfe_rs
b. 
c. 

7. Add the transfers.txt file into the GTFS zip file

8. Run the Navitia Server
-------------------------
a. Clone the following repository to your machine: 
$ git clone https://github.com/shakedk/navitia-docker-compose.git

b. Open git bash at the repository folder and run 
$ docker-compose up
Once all the containers are up, green 'done's appear.


9. Copy GTFS and OSM files to generate the transit graph in Navitia
-------------------------------------------------------------------
Copy the GTFS file (with the transfers file included) and the OSM files into the Navitia server containers. Run the following commands one-by-one in a NEW terminal:

$ docker cp <Files_Folder>/<GTFS-With-Transfers-file>.zip navitia-docker-compose_tyr_worker_1:/srv/ed/input/default/
$ docker cp <Files_Folder>/israel-and-palestine-latest.osm.pbf navitia-docker-compose_tyr_worker_1:/srv/ed/input/default/

The copy process takes several seconds (there's no configrmation for it) and then the graph build takes abour 30-40 minutes. In the mean time, we'll setup the Navitia playground


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
You can view the worker server logs to see if the job completled succefully:

a. Save the worker logs to a fodler on your machine by opening a terminal and running:
$ docker logs navitia-docker-compose_tyr_worker_1 >& workerLogs.txt

b. Open workerLogs.txt and search for "gtfs2ed" or "osm2ed". The logs should show that these jobs are launches and succeded, e.g.:
	Line 25170: [2018-11-06 13:14:41,040] [ INFO] [    1] [        celery.worker.job] Task tyr.binarisation.gtfs2ed[37095a3c-5fbd-494a-9660-0040fb9dffb8] succeeded in 1558.0350151s: None