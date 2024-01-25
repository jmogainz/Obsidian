Architecure
	---
	GLOBAL APPLICATION
	---
	Waypoint Follower
	---
	INPUTS
	---
	GPS: odometry/gps (ekf file navsat_transform, with ublox xed-f9p)
	IMU: data/imu 
	VESCRMP: odometry/rpm
	-
	Topics are remapped in model sdf file
	Same Topics are remapped for receive in ekf.yaml for localization engine
	---
	These information can be provided to the package through `nav_msgs/Odometry`, `sensor_msgs/Imu`, `geometry_msgs/PoseWithCovarianceStamped`, and `geometry_msgs/TwistWithCovarianceStamped` messages.
	---
	Fused sensor data is published by the `robot_localization` package through the `odometry/filtered` and the `accel/filtered` topics, if enabled in its configuration. In addition, it can also publish the `odom` => `base_link` transform on the `/tf` topic.
	---
	OUTPUS
	---
	cmd_vel message needs to be converted to ackermann and shown to vesc plugin
	Twist:
		vel
		angular vel
	Ackerman:
		steering_angle
		steering_angle_vel
		speed
		acceleration
		jerk
	

	
		
	
