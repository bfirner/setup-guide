After you have the software set up on the server (see the Infrastructure guide), follow these steps to get the sensor data working. I will put the ip address of the server as "< server ip >", but you should insert the ip address of your own (or localhost if you are setting up the system on your own computer).

Verify that the aggregator is running
=====================================
1. Telnet to the server IP address at port 7008. Example output:

		bash-4.2$ telnet <server ip> 7008
		Trying <server ip>...
		Connected to <server ip>.
		Escape character is '^]'.
		GRAIL solver protocol

Set up receivers
===================
Steps 1 and 2 assume that you are using the Raspberry Pi receivers -- if you want to run a receiver from a different computer then please run the pip\_sense\_layer code from https://github.com/bfirner/pip\_sense\_layer

1. Program the server IP address on the Raspberry Pis so the receivers send data to the server.
The ip address of the server goes into /etc/owl/pip.confg on the SD cards of the Pis
2. Plug a Pi into power and ethernet and verify that the colored LEDs on the Pi come on
3. Use the aggregator\_example.rb script to verify that the receiver is working. You can fetch this repository with:

		git clone https://github.com/OwlPlatform/ruby-examples.git
             You must have the libowl gem installed
4. Example output:

		ruby aggregator_example.rb <server ip> 7008
		NaN: (phy [1]) 2981 -> 231, RSS:-72.0, Datalength:2 Data:
		Processing some packets!
		NaN: (phy [1]) 2988 -> 192, RSS:-90.0, Datalength:2 Data:
		Processing some packets!
		NaN: (phy [1]) 2988 -> 231, RSS:-59.0, Datalength:2 Data:
		Processing some packets!
		NaN: (phy [1]) 2988 -> 199, RSS:-53.0, Datalength:2 Data:
		Processing some packets!
		NaN: (phy [1]) 2474 -> 231, RSS:-63.0, Datalength:2 Data:
		Processing some packets!
		NaN: (phy [1]) 2474 -> 192, RSS:-92.0, Datalength:2 Data:
The first ID is the transmitter ID and the second is the receiver. In this example there are three receivers running (199, 231, and 192). When you plug in a new receiver check to make sure it is sending data by seeing that packets from its receiver ID are sent to the aggregator.


Verify that the world model is running
======================================
1. Telnet to the server IP address at port 7009. Example output:

		bash-4.2$ telnet <server ip> 7009
		Trying <server ip>...
		Connected to <server ip>.
		Escape character is '^]'.
		GRAIL world model protocol

Run Some Solvers
================
Solvers transform sensor data from the aggregator into data for the world model.
1. First add an object into the world model. An example of this is the make\_new\_entry.rb script from the previously mentioned ruby-examples repository, which creates entries for use with the gwt owl platform demo website.

		ruby make_new_entry.rb <server ip> 7009 Ben Hefei.door.303 2878 "Door 303"

2. Verify that the data was added into the world model with the client.rb script in ruby-examples.

		bash-4.2$ ruby client.rb <server ip> 7010
		Searching for all URIs and attributes
		Found uri "Hefei.door.303" with attributes:
			creation, 1376528502802, 0, Ben: [""]
			sensor, 1376530648379, 0, Ben: ["0100000000000000000000000000000b3e"]
			displayName, 1376530648379, 0, Ben: ["0052006f006f006d0020003300300033"]

Now run some simple solvers
---------------------------
1. Download and run the pip-data-solver to interpret the binary data from the pip sensors.

		git clone https://github.com/bfirner/pip-data-solver.git
		cd pip-data-solver
		cmake .
		make
		sudo make install
		nohup unbuffer /usr/local/bin/owl/pip-data-solver <aggregator ip> 7008 <world model ip> 7009 > pip-data.log &

2. This should have the pip-data solver running in the background, writing log data into pip-data.log. You can follow the log file by typing "tail -f pip-data.log"
3. This has made raw sensor data available to solvers that connect to the world model. Now we will download run the temperature solver.

		git clone https://github.com/bfirner/temperature-solver.git
		cd temperature-solver
		cmake .
		make
		sudo make install
		nohup unbuffer /usr/local/bin/owl/temperature_solver <world model ip> 7009 7010 > temperature.log &

4. Again, you can follow the log file by typing "tail -f temperature.log". You should see messages about the temperature of any entries in the world model. Note that temperature tends to fluctuate between two values if it is in the middle (eg. a real temperature of 29.5 will result in reported temperatures fluctuating between 29 and 30).
5. Now we will download the binary state solver, which provides such information as closed status of doors and wet status of flood sensors.

		git clone https://github.com/OwlPlatform/binary_state_solver.git
		cd binary_state_solver
		cmake .
		make
		sudo make install
		nohup unbuffer /usr/local/bin/owl/binary_state_solver <world model ip> 7009 7010 <config file> > binary_state.log &
The config file is in the binary\_state\_solver/conf/binary\_types.conf. You should edit this file for the kinds of objects that you want to collect binary data for. It defines the name of the solution type for objects in the world model; for instance the line:

		door closed
tells the binary\_state\_solver that anything with a name matching the pattern .*\.door\..* (eg. Hefei.door.303) has a binary attribute that should be named "closed". You can add any number of lines to this file, and if you have flood sensors you should add the entry "flood wet".
6. Verify that these solvers are sending data to the world model:

		bash-4.2$ ruby client.rb <server ip> 7010
		Searching for all URIs and attributes
		Found uri "Hefei.door.303" with attributes:
			creation, 1376528502802, 0, Ben: [""]
			sensor, 1376530648379, 0, Ben: ["0100000000000000000000000000000b3e"]
			displayName, 1376530648379, 0, Ben: ["0052006f006f006d0020003300300033"]
			closed, 1376553799601, 0, binary_state_solver: ["00"]
			temperature.celsius, 1376553901407, 0, temperature_solver: ["403d000000000000"]

Congratulations! You now have a working Owl system with data in it! The next step is to get some visualizations working.
