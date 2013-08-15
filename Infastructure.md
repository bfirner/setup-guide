Welcome to Owl Platform!
========================

This guide will guide you through settings up the software infrastructure of an Owl platform system. There are two main software components to the system, the aggregator and the world model.

The Aggregator
==============
With a running sensor system you will have many, many sensors. It is impractical to deal with each sensor individually, so the aggregator "aggregates" sensor data in one location. The aggregator runs as a network service with sensors sending it data on one TCP port and software requesting that data running on another port. You can download the aggregator with git:

		git clone https://github.com/OwlPlatform/aggregator.git

Follow the instructions in the README.md file in that repository to build the aggregator. The aggregator is programmed in Java, so you can also easily copy the aggregator from one machine to another if you have alrady built it. The aggregator depends upon a Java installation and an the Apache maven tool.

By default the aggregator accepts sensor information on port 7007 and responds to requests for sensor data on port 7008.

The World Model
===============
The world model is a tuple store kind of system, with named objects and their lists of attributes. Attributes are a bit like files on your file system; each attribute has a name, data, a creation date, and the name of the person (or program) who created that attribute. Attributes in the world model also have an expiration date that tells you when the data is no longer valid. Timestamps in the world model are the number of milliseconds since 1970 (Unix time). In this file system example, the folder would be a named object. Just as a folder contains multiple files a named object contains multiple attributes.

You can download the world-model like so:

		git clone https://github.com/OwlPlatform/world-model.git

Follow the instructions on https://github.com/OwlPlatform/world-model to build the world model. You probably want to just use the sqlite3 world model implementation, so the steps to build this are:

		cmake .
		make sqlite3_world_model_server

Then copy the bin/sqlite3\_world\_model_server file into /usr/local/bin/owl

You will need a fairly recent version of g++ (at least 4.6 I believe), sqlite3, and cmake to build the world model. The world model page on github has more information.

Please note! The sqlite3 world model will create a sqlite3.db file in the directory where you run it. This will contain all of the world model information. If you run the program from a different directory it will not be able to see all of those existing entries. You can copy the database file (and its temporary files named world\_model.db-*) to move the data to a different directory.

By default the world model will accept new data on port 7009 and will disseminate data on port 7010.
