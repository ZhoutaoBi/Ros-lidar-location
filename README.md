
# Copyright (C) [2024] by [ZhoutaoBi] 

# Project description
	The project is the deployment of Lidar in the G unmanned aerial vehicle of the National College Students Electronic Design Competition in 2023. 
	It was created by three members of AIENSS of South China Agricultural University. 
	If you have any questions, please contact us at bizhoutao@yeah.net

# ros-location
	The program use the Lubancat1S with linux and Lidar by hector-mapping algorithm,
	Lubancat 1S, install the simplest system, ssh commond

# How to use the program

	Whatever you use any board,you also need follow the steps like 1,2,3,5,6
	the hector-mapping package you need to download from GitHub (the correct version of the package),
	different ros need different version of the hector-mapping package,every board have their own user manual,
	you need to follow the steps in their manual like wifi,serial port and so on.
	In my catkin_ws(GitHub\ros-location\catkin_ws\src),the finish program is (GitHub\ros-location\catkin_ws\src\topic_example\launch\bi.launch)

		1:GitHub\ros-location\catkin_ws\src\hector_slam-melodic-devel
		The folder is the hector-mapping package,we only use the (src\hector_slam-melodic-devel\hector_mapping),
		other folders are not used.

		2:GitHub\ros-location\catkin_ws\src\lsn10p
		This folder is about the basic configuration of radar

		3:GitHub\ros-location\catkin_ws\src\topic_example
		This folder is about the 
			1:open serial port 
			2:radar data processing and mapping data publishing 
			3:communication between the Map coordinates and stm32 by serial

		4:GitHub\ros-location\catkin_ws\src\robot_upstart
		This folder is about the auto start of the program when you turn on the power

	some important change tips:
		1: Real-time updates about coordinates
			If you use the hector-mapping, you'll notice that there's some coordinate delay.
			Since we need to use the lidar coordinates and the optical flow coordinates of the UAV for data fusion, 
			it is necessary to improve the real-time performance of the map coordinates, 
			but the output speed of the built-in map coordinates will cause congestion. 

			So I made the following changes:
			GitHub\ros-location\catkin_ws\src\hector_slam-melodic-devel\hector_mapping\launch\mapping_default.launch

			at line 7: arg_name="scan_subscriber_queue_size" default="1"
			I make the default = 1.The default value was originally 10, but I set it to 1 so as not to cause congestion, 
			the old data will be replaced directly, and only the latest coordinate will be stored. 
			The disadvantage is that it may lead to poor coherence of the data and some distortion of the waveform, but it is not harmful.

		2: Design of communication protocol with stm32


			/*****************************************/
			I suggest that you can write a new protocol to communicate with stm32 by yourself.
			This protocol of the project is modified from the ROS balancing cart routine.
			Due to the urgent time at that time, many contents did not add comments, 
			but directly modified the parameters to meet the protocol. Therefore, 
			I suggest that you design a protocol directly and rewrite it directly, which is simpler and faster.
			/*****************************************/


			made the following changes:
			GitHub\ros-location\catkin_ws\src\topic_example\src\mbot_linux_serial.cpp

			The sned communication protocol is:

			Header frame: bug[0] = 0x55 buf[1] = 0xaa

			data frame:
				buf[2] = length; //buf[2]
				buf[3]buf[4] = position.x
				buf[5]buf[6] = position.y

			ctrlFlag frame:buf[7]= 42 // 42 is the control flag

			buf[8] :
				GitHub\ros-location\catkin_ws\src\topic_example\src\mbot_linux_serial.cpp line 136 
				unsigned char getCrc8(unsigned char *ptr, unsigned short len)

				GitHub\ros-location\catkin_ws\src\topic_example\src\mbot_linux_serial.cpp line 69
				buf[3 + length] = getCrc8(buf, 3 + length);//buf[8]
			
			buf[9]:0x0d
			buf[10]:0x0a


			The recived communication protocol is:

			Header frame: bug[0] = 0x55 buf[1] = 0xaa
			length = buf[2]; 
			buf[10] = getCrc8;

			more information about the protocol can be found in the stm32 code.


# init the ros environment and ros package
1、open wifi
	sudo nmcli radio wifi off			
	sudo nmcli radio wifi on			
	nmcli dev wifi list			
	sudo nmcli device wifi connect "user_name" password "xxxxx" ifname wlan0 

2、ros is installed with one click

	wget http://fishros.com/install -O fishros && . fishros

3、Create workspace

	mkdir -p ~/catkin_ws/src
	cd ~/catkin_ws/src
	catkin_init_workspace
	cd ~/catkin_ws/
	catkin_make
	source devel/setup.bash


4、Open serial ports 3 and 5 // The two serial ports are connected to radar and flight control respectively

    /*
    *
    * SUGGESTION:This step you can followed by the user manual of the board you use.
    *
    */
	/boot/uEnv/board.txt (board is the name of the board you use.)
	restart		//Enter "sync" on the terminal and then power off the device
	ls /dev/tty* 	//Check whether the function is enabled successfully, for example, /dev/ttyS3


5、Test radar reading data,Place the lsn10p package into the src workspace
	/* Operate according to radar documentation */	
    // the example of the radar data is as follows:
	catkin_make -DCATKIN_WHITELIST_PACKAGES=lslidar_msgs
	catkin_make -DCATKIN_WHITELIST_PACKAGES=lslidar_driver
	roslaunch lslidar_driver lslidar_serial.launch	// Serial port print data
	/*
    *Various packages will be missing when installing, because it is a simple version of ros, according to the error csdn!
    */
	
6、Go to GitHub to download the corresponding hector-mapping package and put it in src
	'catkin_make' your workspace

7、test the finish

