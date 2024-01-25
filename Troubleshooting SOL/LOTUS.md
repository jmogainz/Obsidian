Current Status

TODO
Code commands between bs and bs receiver
Fix facing east requirement

Sim 
	Dynamic (no longer static map to odom transform)
	Car must be spawned facing east
		Probably due to the imu sensor gazebo plugin (relative, not absolute heading) 
		Possibly try absolute heading

Real
	Dynamic (no longer static map to odom transform)
	Car must start facing east due to map frame initialization
	Currently map frame is being initialize with -y being straight and +x being left (INITIALIZED FROM CURRENT HEADING)
	Our waypoint to map in base station receiver sets up the pose for a facing east map frame (due to lat long meter calculation using UTM)
		Options to fix this include modifying x, y position to match the initialized map frame
			Possibly take example north facing 1.57 orientation, find east (which is now -1.57, considering north is 0 in initialized frame), find angle from east that the waypoint is (azimuth? or atan2(x,y)) and add it to this east (-1.57) value. This gives the heading from our initialized orientation (north), or the angle to the waypoint. From their, we can use trig to determine the x, y values of this new angle (using the angle in conjuction with the hypotenuse (distance to waypoint)).
			Waypoint orientation would be how calculated waypoint heading
		Options to fix this include finding manual utm to map tf2 ros functions to create a pose in space and then send this like usual to waypoint follower