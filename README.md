AGV_Localization
================


					Odometry, IMU and Robot Localization Packages

<0>.About <robot_localisation> package
0.1.The package is basically made for <PR2> bot as it uses namely 3 sensors value
	0.1.0.Odometry_data<odomX>
	0.1.1.Imu_data<odomX>
	0.1.2.Visual_Odometry<voX>	


<1>.How Robot Pose EKF works (http://wiki.ros.org/robot_pose_ekf)

1.1.Pose interpretation--< Calculation of pose w.r.t. to relative usage not absolute>

	All the sensor sources that send information to the filter node can have their own world reference frame, and each of these 		world reference frames can drift arbitrary over time. Therefore, the absolute poses sent by the different sensors cannot be 		compared to each other. The node uses the relative pose differences of each sensor to update the extended Kalman filter.

1.2.Covariance interpretation--< Use of covariance for both pose and it derivative (i.e. velocity) >

	As a robot moves around, the uncertainty on its pose in a world reference continues to grow larger and larger. Over time, the 		covariance would grow without bounds. Therefore it is not useful to publish the covariance on the pose itself, instead the 		sensor sources publish how the covariance changes over time, i.e. the covariance on the velocity.

1.3.Timing--< Use of Interpolation>

	Imagine the robot pose filter was last updated at time t_0. The node will not update the robot pose filter until at least one 		measurement of each sensor arrived with a timestamp later than t_0. When e.g. a message was received on the odom topic with 		timestamp t_1 > t_0, and on the imu_data topic with timestamp t_2 > t_1 > t_0, the filter will now update to the latest time 		at which information about all sensors is available, in this case to time t_1. The odom pose at t_1 is directly given, and 		the imu pose at t_1 is obtained by linear interpolation of the imu pose between t_0 and t_2. The robot pose filter is updated 		with the relative poses of the odom and imu, between t_0 and t_1. 
	
<2>Nodes
2.1.robot_pose_ekf
	robot_pose_ekf implements an extended Kalman filter for determining the robot pose.

2.2.Subscribed Topics
	2.2.1.odom (nav_msgs/Odometry)
		2D pose (used by wheel odometry): The 2D pose contains the position and orientation of the robot in the ground plane 			and the covariance on this pose. The message to send this 2D pose actually represents a 3D pose, but the z, roll and 			pitch are simply ignored. 
	2.2.2.imu_data (sensor_msgs/Imu)<Use of Yaw as relative angle>
		3D orientation (used by the IMU): The 3D orientation provides information about the Roll, Pitch and Yaw angles of the 			robot base frame relative to a world reference frame. The Roll and Pitch angles are interpreted as absolute angles 			(because an IMU sensor has a gravity reference), and the Yaw angle is interpreted as a relative angle. A covariance 			matrix specifies the uncertainty on the orientation measurement. The robot pose ekf will not start when it only 		receives messages on this topic; it also expects messages on either the 'vo' or the 'odom' topic. 

2.3.Flexiblity of Number of sensors and their flexiblity of data sending
	The robot_pose_ekf node does not require all three sensor sources to be available all the time. Each source gives a pose 		estimate and a covariance. The sources operate at different rates and with different latencies. A source can appear and 	disappear over time, and the node will automatically detect and use the available sensors.

<3>.Migration from robot_pose_ekf (http://wiki.ros.org/robot_localization/Tutorials/Migration%20from%20robot_pose_ekf)
	3.1.Covariances in source messages<Setting high covarience to ignore the sensor>
		 For robot_pose_ekf, a common means of getting the filter to ignore measurements is to give it a massively inflated 			 covariance, often on the order of 10^3. However, robot_localization allows users to specify which variables from the 			 measurement should be fused with the current state.

    	3.2The differential parameter<Differential Integration>
		By default, robot_pose_ekf will take a measurement at time t, determine the difference between it and the measurement 			at time t-1, transform that difference into the current frame, and then integrate that measurement. This cleverly 			aids in cases where two sensors are measuring the same pose variable: as time progresses, the values reported by each 			sensor will start to diverge, eventually causing the filter to jump back and forth between the measured values. By 			carrying out differential integration, this situation is avoided and measurements are always consistent with the 			current state.

	3.3For robot_localization, the preferred method is to feed it velocity information and let the filter integrate the values 			using its kinematic model. If this is impractical (e.g., for measurements from other packages over which you have no 			control), then users can use the xxx_differential setting. By setting this to true for a given variable (the default 			value is false), robot_localization integrates the pose data for that sensor in the same manner as robot_pose_ekf. 			Users wishing to integrate GPS data should take care, though: setting the differential parameter to true for data 			coming from, for example, navsat_transform_node will result in the loss of the "global" nature of the measurement. In 			other words, each measurement will not be fused absolutely and therefore will still drift from the true GPS track. 			Users wishing to fuse GPS data should make sure that its differential parameter is set to false. 

<4>


