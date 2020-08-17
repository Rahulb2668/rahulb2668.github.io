@page codebrief Code in brief
@tableofcontents
@section func Functions

@subsection mqtt MQTT Broker:
Data will be populated in the MQTT broker. Raspberry pi using mqtt client subscribes to the respective topic and gets the payload and stores it in the sql database
Mosquitto is broker used.(for now)

@subsubsection mqttfunc MQTT Functions:
In this process the raspberry pi will subscribe to a specific topic and store the payload in  SQL database. Raspberry pi subscribes and publishes specific topics from and to the broker.
Raspberry pi subscribes to topic agile-bot/Rack_ID/#, gets all the data and stores it in the sql database.
ie, if the broker publishes data over the topic ‘agile-bot/123456/194564/orders/807c9d90-5635-406a-9781-95362b1b9406/foodtype’.It's like ‘agile-bot/rackid/boxid/order/orderid/foodtemp’, master will be subscribed to all topic and master get the published topic and message, splits the topic, stores in list and get the last value of list  ie, here ‘foodtemp’, which is the also the name of  the column in the database, so using sqlite3 , master updates the the respective column of the database. Similarly for all the topics published master updates the database.Master takes the order details from the server once the order is confirmed. The details consist of order ID, OTP for both loading and unloading. The data from the server will also include some other details like temperature, the food to be maintained, type of food.SKU’s etc..
@note Python’s Mqtt library Paho.mqtt is used.

@subsection uart UART program:
Master communicates with the slaves(boxes) via RS485 communication. Where each packet of the data sent will be having a communication ID, command ID, data and LRC. Each slave connected in the bus gets the packet sent on the bus but responds only if the communication ID matches.Master sends and receives the data through the master mispigy. Data from master is sent to master mispigy via UART_A1 and Mispigy transmits it to the bus via UART_A0 through RS485. 

@note Packet definition and details are on the Packet_definition.xlsx

@subsection keypad Keypad program:
Its continuously running program which interfaces the keypad with the master. This program records the key press and stores it in a mmap file which is accessed by other programs. This program is used to collect the OTP, (for now, more will be added).This works on the basis of the interrupt.
Eg: GUI using the keypad program to display the key pressed.

@subsection gui GUI :
Gui is developed with the PyQt5 python package. 
The main Window consists of two layouts one is  the unique rack for placing the order and the other for entering the otp. A label for mentioning the internet connectivity, a label for showing the rack Icon and a label for mentioning the ‘welcome to smart rack’ is added in the main page. \n
There are other windows like:
- Qr Window :  Window displaying the qr for both loading and unloading mode.
- Unloading Window: Displaying the text for the user about his order.
- Loading Window: Displaying the food and respective slots the food has to be placed .
- Select Box Window and Action Window: A Window for debugging and maintenance .


@section programs Programs
@subsection Home_Page
This python script is the main GUI window code. Pyqt5 python module is used to create the gui. This window consists of rack unique qr and the enter otp layout.This script consists of Mqtt client methods as well as Gui methods. \n
![Home Page](doc_Images\Home_Page.png "Home Page") \n

Main parts of this scripts are: \n
- Mqtt Client methods
- Home Window
- Maintenance Window
- Offline Window
- Rack Configuration thread
- Rack initialization thread

***Basic Functioning*** :
The rack can be operated via qr code and also via otp. The rack qr is provided for the user to place orders. The qr consists of the url for the rack. The rack is also operated by entering the corresponding otp given to the user for the order they placed.
When an order is placed the broker sends a topic ‘userotp’ with the otp as payload. Master subscribes to the topic and updates the database with the new otp for the slot. As the broker publishes the ‘userqrshowflag’ or ‘deliveryqrshowflag’ master displays qr in the hmi with the otp as encoded information. User scans the qr and then if the otp is verified by the broker then master publishes ‘userqrverifiedflag’ or ‘deliveryqrverifiedflag’ , when this flag is received, master collects all the details of the order from the database and opens the respective slots accordingly.

@subsubsection main Method : main()
Every Qt program runs in an event loop, where the actions done by the user on the GUI, like button press, mouse click all are termed as signals and all this can be connected to different slots (functions). The eventloop waits for the signals, processes the slot and again continues the loop.
In the main function we are creating an object for QApplication() class of Qt which is the event loop.Then we make an object for the main class in the script Home_Window(). The action is recorded in the log file using logging.info method.

@subsubsection home_window Class Home_Window
In the class for each window like QR window, unloading window, loading window, settings window  specific signals are created which connect to the unique slots when emitted.When an object is created for a class, the init function is automatically called.

@subsubsection ui Method : self.Ui()
The Ui method calls the two main other methods for the Gui, the widgets method and the layout method.After the widgets and layouts are called then show function is called which shows the gui.

@subsubsection widgets Method : self.widgets
This method initializes all the widgets used in the home screen of the Gui.
The widgets used in the home page are:
- QLabel for text “Welcome to smart Rack”
- QLabel for the rack icon and the Qr Code
- QLabel for network label,place order and pick order text
- QLineEdit for OTP
- QPushButton for resetting the text space

@subsubsection layouts Method : self.layouts
The home page consists of a self.mainLayout, which has two sub layouts: self.textLayout and self.bottomLayout. The bottomLayout have two groupboxes: self.leftGroupBox and self.RightGroupBox which holds the widgets layouts like pickoderLayout and placeorderLayout. 
After all the layouts are created , it is set as the layout of the window with self.setLayout() method.

@subsubsection mqtt_client  Mqtt Client Methods:
Broker is the mqtt broker using,  port is the mqtt port and the refresh_time is keepalive time.
The main functions of the mqtt client is to publish and subscribe to the topics to and from the broker.For Mqtt communication Paho Mqtt Python module is used.
In MQTT a publisher publishes messages on a topic and a subscriber must subscribe to that topic to view the message. For the rack there is aws server providing services messages and details about the order confirmed. The master pi has to subscribe to the topic it publishes to the broker and get the details from the broker as mqtt topics and messages.The core of the client library is the client class which provides all of the functions to publish messages and subscribe to topics.Here paho mqtt client module of python is used. First of all an instance of the client is needed i.e. by initializing the mqtt.Client.The security type is made as tls.
In Mqtt there are certain call back functions which are automatically invoked when a certain task happens, examples, client.on_connect etc… we have to define the function and attach the function to the callback. \n \n

***Method : self.Connect()*** \n

broker :’ www.mqtt.agilebot.in’ \n
Port : 1883 \n
Refresh_time = 60 \n

To publish data to the broker we have client.publish method, to subscribe we have client.subscribe() method. This methods are in self.pub_to_broker and self.sub_from_broker methods. \n
As mentioned there are many callback functions:
- Event Connection acknowledged Triggers the on_connect callback
- Event Disconnection acknowledged Triggers the on_disconnect callback
- Event Subscription acknowledged Triggers the  on_subscribe callback
- Event Un-subscription acknowledged Triggers the  on_unsubscribe callback
- Event Publish acknowledged Triggers the on_publish callback
- Event Message Received Triggers the on_message callback
- Event Log information available Triggers the on_log callback \n
When a client issues a connect request to a broker that request should receive an acknowledgment.The broker acknowledgement will generate a callback (on_connect) , when a data is published on a topic successfully on_publish is triggered, on_disconnect is triggered when the client disconnects from the broker and on_message callback is called when a mqtt topic is published on the broker. \n .

***Method : on_message()*** \n
On_message is paho client mqtt callback method, which is called when a topic is published on the broker.
Whenever a  topic is published on the broker,  the on_message method is triggered and the payload is parsed into topic and message. The topic is splitted and stored in a list and the last value of the list the column name of the database or certain action or flag, if its a column name, using the column name and sqlite3 python library the message is written to the database. On the other hand if it's a unique action or flag like ‘Offline’, ‘Onmaintenance’, ‘userqrshowflag’ etc …  the slots for the action are connected by emitting the certain pyqtSignal declared in the condition.
On_message function mainly consists of a main ‘if else ’ condition, it first checks whether the topic published is having a length more than four keywords, if true then its an action or flag related to an order else if false the topic is for the master or response of the master.
When a topic is published in the broker for an order, the on_message function checks if the slave device_id  is already in the database, or if it's a new slave then it's added to the database. If the slave is already present in the database, then order details are updated in the database. Else if the slave device is a new one, a new row with the device is inserted to the database and a communication id is assigned.

@note  If a new slave id is received then, along with updating the slave_id in the database, the communication id is broadcasted.\n .

***Actions and Flags*** \n
When an order is placed, the broker sends an ‘userotp’  topic with payload as the user otp,
Master stores the otp in the database by the snippet shown below,
This snippet is performed such that when the topic is not an action or flag.
After the order is placed, as the user presses the collect food option in the website or application of smart rack, ‘userqrshowflag’  is published, as the flag is received by the master the self.qrsignal is emitted to the slot to display the qr window, the qr window displays the qr with information of the otp received. After the user scans the qr , if the otp is correct then the broker publishes another flag of ‘userqrverifiedflag’  when this flag is arrived, the otp for the box is taken from the database and passed to the self.accept method. This is the same for the loading or delivery mode.

@subsubsection accept Method : self.accept()
This method is the main action method which triggers the loading and unloading action and opens the loading and unloading window. When an otp is passed to the method, it checks whether the otp is for loading operation or for unloading operation ie,‘userotp’ or ‘deliveryotp’ ,This is done with check_who function, which returns deliveryflag or userflag according to the type of otp. The otp type is checked in the sql local database.
If an otp is passed , it checks whether the otp is ‘AB*#AB’ if yes it closes the application, if the orp is *0000#  then the signal for the opening setting window is emitted. This settingsignal is connected to a slot which opens the setting window. If the both conditions are wrong then otp is made to integer and passed to the check_who method where the master checks whether the otp is for loading operation or unloading operation. check_who method returns flag for loading operation it returns ‘delivery_flag’ and  for unloading operation ‘user_flag’
If the otp is loading otp, then a dictionary with boxno and food name are created and passed to the placefoodwindow.py. Here the box numbers are taken based on the sku_ids and are created such that for one skuid, the corresponding boxes. And if it's a user otp then the user name and food list is taken and passed to the userwindow.py along with boxno’s. If the otp is not found in the database an invalid otp message box is displayed.

@note - In the init method self.Connect is called.\n - The accept method is functioned only when the activeflag is True.

@subsubsection showqr Method : show_qr()
When a qrshowflag is published on the broker for either loading or unloading master displays the qr
Window. Here an object is made for the class Qr_show in QRshow python script. Then showingqr topic is published and then it is logged. After the window opens the signal connected to the slot is disconnected 


@subsubsection setting Method : show_setting()
When otp entered is ‘*0000#’ or the topic published is ‘setting’, then this slot is called. Here an object for the selectBox class is created which opens the setting window. This is basically a debug or testing window.

@subsubsection showunloading Method : show_unloading
As the QR code or otp is verified the window is shown with the name of the user and the food items ordered. This slot is connected to a pyqtSignal ‘unloadsignal’ which is disconnected after the slot is executed. After the commands for opening the slot are sent, master publishes the ‘delivered’ topic for each slot for that order.


@subsubsection showloading Method : show_loading
When the qr or otp for loading is verified an window is shown with food and the respective boxes for the food to be placed.This window is the loading window. The show_loading slot is connected to a signal called ‘loadsignal’ which is disconnected after the slot is executed. This slot has a dictionary of the food names and the boxes as the argument which is passed to the ‘Loading_Window’  class. After the food is placed in the boxes, the boxes have to be configured based on the metadata of the order. This is done by a thread class ConfigWorker, this works as a different thread from the parent thread, which helps in sending the metadata to the slave without freezing the GUI and after the command and metadata are sent to the boxes the thread is destroyed.ALong with configuring the boxes, master publishes ‘loaded’  topic to the broker.
In the ConfigWorker class box list is passed to the configure_rack method defined in the serialpacketisation.

@subsubsection showmaintenance Method : show_maintenance()
When a ‘Onmaintenance’  topic is received the activeflag is turned to False and the maintenance window is displayed in the GUI. In this method an object for the maintenanceWindow class is created. \n

@subsubsection maintenanceWin Method : Class maintenanceWindow()
In this class a QWidget is created in which a label is used to display an image. When this window is displayed the command for turn off all the leds are sent. 
![Maintenance Window](doc_Images\Onmaintenance.png "Maintenance Window") \n
This is the same for offlineWindow class  also. \n
![Offline Window](doc_Images\Offline.png "Offline Window") \n
The ‘activate’ and ‘online’ topic is published to get the rack back to active state.

Whenever the rack is booted up the communication id is broadcasted for each slave box in the rack. This is done using another thread class:


@note For more info @ref Home_Page.py

@subsection Loading_Window
A dictionary of food names and respective boxes are given to the class. A window with food names and the boxes to place are displayed on the HMI as well the boxes are opened one by one. The commands to open the boxes are via a Qthread class commWorker(). \n
![Loading Page](doc_Images\loading.png "Loading Page") \n

At first the populate() method boxes and food names are made as two lists. Then in the window a label is created and a text is added to the label such that the ‘Place {Food Name} in the {Box_no }’ and it is displayed. This QLabel is created in the Layout() method of the class. After the labels are displayed the commWorker thread is started which starts sending the open command to the boxes one by one.  If there are 3 biryanis(Same skuids), then 3 different  boxes have to be opened.This window has a scroll layout which automatically appears when the list of items exceeds the screen size.

@note For more info @ref Loading_Window.py

@subsection Unloading_Window
This window is for displaying the username and order items while delivering the food. A QWidget is created in which a label is made which has a text like ’ Your Order for {food name }’ is ready.\n
![Unloading Page](doc_Images\userwindow.png "Unloading Page") \n
As the window is displayed in child thread the commands to open the boxes, turn off the heater and put the box the user mode are sent and then cloudUpdatesig is emitted and as well finished signal is emitted.

@note For more info refer Unloading_Window.py

@section serialwithpacketisation
This python script is for the communication between the master box and slave boxes,
It sends and receives data to and from the data bus
@subsubsection send_to_bus
This method takes the arguments to be sent and returns True when the communication is completed and returns False when error happens. This method takes the arguments packs int into a list called packet and pass it to the send_packet method

@subsubsection send_packet
This method takes the data to be sent and makes it in a packet in the form of ‘[{,( ,boxid, command1, command2,LRC, ),} ]’. After packetizing the port_write method is called to write the data over the bus. The port_write method returns true after completion of one packet.If it returns true, then inside the send_packet() method the command is checked and if the command sent is less than 4 then the method calls the get_reply() method to get the reply from the slave and it stores the reply to the database.

@subsubsection port_write:  This method takes the packet to be sent and sends the data over the bus as bytes.
Time delay of 0.18 ms is given after a byte.

@subsubsection get_reply: This method takes the slave boxid and the command sent. This method gets the reply from the slave and checks if the reply is valid from the slave. The response of the slave is  made like ‘[{( boxid, cmd, response , LRC, )}]’ . The function checks the LRC, does the device id and cmd matches , if it does it returns the data and returns true.

@subsubsection computeLRC: This method is to compute the LRC of the data. 

@subsubsection broadcast_id:
This function takes two arguments one is the device_id and the other is the box_id,. This method is called to broadcast_id the assigned communication id to the new device installed to the rack;
Once the communication id is assigned ,reassigning is not done to that device. 
@note For more info @ref serialwithpacketisation.py
@section Initialize_box
This code runs for one time. It subscribes to the topic of the rack from the broker and also configures the rack with all the data it has in the database for one time.The rack is made to loading mode first by sending command of (0,9,0) then the box_id and dev_ids are taken from the database and then communication id is broadcasted with ‘boradcast_id’ method and then command is sent to make the led turn red and then back to green.
@note For more info @ref initialize_rack.py

@section Database
A class Database is made with all the sqlite3 database operations. This class is used throughout the application.There are two types of topics, one is the master data and the other is the slave box data, which are classified on the basis of the length of the topic.
When a topic is published in the broker for a slave, the on_message method in Home_page calls the Database methods like write_boxdata_db_dev_id or write_masterdata_db.
@subsubsection write_boxdata_db_devid
This method is called from on_message callback, column dev_id and the value to be updated in the database is passed to the function. In the function it checks whether the dev_id is already in the database if the dev_id is already in the database the value is updated in the database and if the dev_id is not present in the database then the a new row is inserted with new  dev_id and the at the same time assign_box_id() method is called which broadcasts the communication id and the dev_id.

@subsubsection write_masterdata_db
This method updates the database for the master datas.There are master data like masterid, master location, current power status etc… all these details are written to the database with write_masterdata_db method. It takes three arguments one is the database using, the column to be updated and the value. 
@subsubsection write_boxdata_db
Slave box datas like orderid, item name etc… all these details are written to the database with this method. It takes three arguments one is the database using, the column to be updated and the value. 
The device id is a 4 byte id, so whenever master communicates to the slave, either it has to use this 4 byte id or it can use a unique comm id for addressing the slave, this is done in this method. It takes all the device id and assigns it a unique comm id and updates the database. After this all the communication between master and slave is using this box_id. After box_id is updated the new box_id and device id is broadcasted to the slaves to store the new comm id to the slave box.
Similarly there are functions to get data from the database, like the temperature, order details etc.
@subsubsection get_all_for_otp 
Function gets all the data for a otp.

@subsubsection get_column_from_db 
This method retrieves specific column from the database for a specific box

@subsubsection check_who
This method is to find the entered otp is loading otp or unloading otp and return flag accordingly. It takes the otp via arguments firstly and try to get all the details with  the otp as  ‘deliveryotp’ column in the database , if the data is None, it means the otp is not in the ‘deliveryotp’, so it try to  get the data from db with otp as ‘userotp’. If data still None, then otp is invalid, else return the flag.

@subsubsection reset_database
This method is to clear the database after the completion of the order.
It deletes the complete row and inserts again with boxid and dev_id.

@note For more info @ref Database.py

@section selectBox
A window with all the 23 boxes as buttons is made. When you select one box, an action window is opened where you can test individual operations like solenoid test, encoder reading, temperature reading etc… \n
![selectBox Page](doc_Images\selectBox.png "selectBox Page") \n
When you select a box the box_id is passed to the action window with the help of sender()  function in pyqt.
@note For more info @ref selectBox.py
@section action
Action window is incorporated with QPushButtons for operations like testing motors, testing heater etc..
When an action is selected the doAction() method is called where the task to be done is taken from the name of the pushbutton, using sender() pyqt method. \n
![action Page](doc_Images\action.png "action Page") \n
The action is stored in variable ‘act’ and with if else if loop the condition is checked and the action is performed.
@note For more info @ref action.py