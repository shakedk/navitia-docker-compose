Deploy Multiple instances on one Navitia Server and moving the Navitia graph between servers
============================================================================================
Let's say we want to have 2 coverages (Navitia's instances) on one server, so by running one custom "docker-compose up" command
we have 2 different covereges, each genrated by a diffeent GTFS file and can be queried concurrently.

This guide will explain how to create 2 covereges using the same OSM file for all 2 and a different GTFS file for each one. 
It also explains how to move Navitia lz4 graph between coverages to quickly create a new coverage with already processed data.

How many coveregaes should we put on one server?
================================================
After some resaerch, for a server with 8GB it is better to have only 2 coveregaes with 2 threads each for each covereage (kraken). With 3 covereges, even one thread per kraken, there isn't enough memory and jormungandr (the web server) announce one of the covereges to be "dead" after several queries.
According to Navitia's team, the "dead" region can be fixed by extending the jormungandr-kraken timeout can be extended, but is it not an envrionment variable, so it has to be modified inside the docker files:

CIRCUIT_BREAKER_INSTANCE_TIMEOUT_S = 60  # the circuit breaker retries after this timeout (in seconds)
See https://github.com/CanalTP/navitia/blob/dev/source/jormungandr/jormungandr/default_settings.py#L130


1. Create a custom docker-compose file with a new coverage
==============================================================
The custom file, let's call it "docker-israel-custom-instances.yml", should exist in the same folder as the regular navitia-compose folder.
In this file we specify the custom instance in addition to the default instance that is provided with the rgular docker-compoe.yml file.


These are the file contents to be altered for a new coverage - replace <covereage name> with the desired name for this coverage

=====custom docker-compose contents====
version: '2'

services:

  kraken-<coverage-name>:
    image: navitia/kraken:latest
    environment:
        - KRAKEN_GENERAL_instance_name=<coverage-name>
        - KRAKEN_GENERAL_database=/srv/ed/output/<coverage-name>.nav.lz4
        - KRAKEN_BROKER_host=rabbitmq
        - KRAKEN_GENERAL_nb_threads=2
    volumes_from:
      - tyr_beat:ro
    expose:
      - "30000"

  jormungandr:
    environment:
      - JORMUNGANDR_INSTANCE_<coverage-name>={"key":"<coverage-name>","zmq_socket":"tcp://kraken-<coverage-name>:30000"}

  instances_configurator:
    environment:
      - INSTANCE_<coverage-name>=
=====custom docker-compose contents====

2. Running Both default and Coverage instances (data providing is after this)
=============================================================================
a. If you "docker-compose up" you only get the deafult coverage. Run:

"docker-compose -f docker-compose.yml -f docker-israel-custom-instances.yml up"

b. After a minute or so, you can access both covereges by going to "http://localhost:9191/v1/coverage/default" and "http://localhost:9191/v1/coverage/<covereage-name>"

Unless you already provided data to the covereges, you will see "no data" messge

3. PROVIDING DATA - RAW & PROCESSED
====================================
In order for Navitia to work, one need to provide it GTFS & OSM data. Once the data is provided, the server generates a graph that is used for calcualtion and query answers, this graph is called <covereage-name>.nav.lz4, e.g. default.nav.lz4

If you don't have a prepared graph, Look at "README - how to deploy Navitia for Israel.txt" - go to "9. Copy GTFS and OSM files to generate the transit graph in Navitia"

If you do have a graph, look at "README - how to deploy Navitia for Israel.txt" - go to "RUNNING NAVITIA WITH EXSITING GRAPH"

4. Restart the Server with the cusom file for changes to take place
===
































