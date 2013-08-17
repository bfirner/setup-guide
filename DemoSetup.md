GWT Owl Demo Page Setup and Troubleshooting Guide
=================================================

This guide will help you set up and troubleshoot the tomcat webapp for the system. For simplicity we will assume that you have already built or obtained the owldemo.war file. I will assume that the world model address is < server ip > and that the region you are setting up the demo for is named < region >.

Set Up World Model Entries
--------------------------
The demo page was built to display certain kinds of sensors, namely door and flood sensors. It finds these in the world model by looking for objects whose names match the POSIX regex pattern ".*\.door\..*" or ".*\.flood\..*". Those patterns match any names that look like "A.door.B" or "A.flood.B" where A and B can be any string.

1. We can use the make\_new\_entry.rb script from the ruby-examples repository to create entries for use with the gwt owl platform demo. For example:

		ruby make_new_entry.rb <server ip> 7009 Ben <region>.door.303 2878 "Door 303"
will create an entry named "Hefei.door.303" in the world model (perhaps the door to room 303), will tell the demo to use sensor 2878 for sensed data, and will name it "Door 303" on the demo web page. Now that that is done we can test the demo.

2. After adding an entry you must restart tomcat if it is already running. See the next section for details to restart tomcat.

Best Case: Everything Works
---------------------------
1. Place the owldemo.war file in the webapps directory of your tomcat installation. On Debian systems this may be /var/lib/tomcat7/webapps, but depending upon your version of tomcat and distribution it may be /var/lib/apache-tomcat/webapps or other variations.

2. Restart tomcat. On a Debian system run 

	sudo service tomcat7 restart
On a linux distribution with the older rc init scripts the command may be

	sudo sh /etc/rc.d/rc.tomcat restart

3. Go to

	http://localhost:8080/owldemo/?q=< region >
in your web browser. The sensor should show up on the page. To get it to change state and show a temperature value you should run the binary_state_solver and temperature_solver. Descriptions for doing that appear in the SystemSetup guide.

Using a Non-Localhost World Model
---------------------------------
You can configure the behavior of you GWT demo by changing some values in the web.xml file of your deployment. From the webapps directory of your tomcat installation it is located in

	owldemo/WEB-INF/web.xml
There are several configuration variables.

wm.host
-------
Configures the host address of the world model that the webapp will connect to.

wm.port.client
--------------
Configures the port of the world model that the webapp will connect to for data requests.

wm.port.solver
--------------
Configures the port of the world model that the webapp will connect to for modifying the world model (changing the demo object names).

sms.username
------------
TODO

sms.password
------------
TODO

