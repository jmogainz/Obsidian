Path Tracking Goal:
	- Compute vehicle control commands so that it can follow a previously planned path
	- Resulting control input should minimize the difference between the planned and actual path
		- Uses lateral distance and vehicle heading
			- Heading is degrees from magnetic north (compass direction)
				- GPS direction is calculated from last two locations where object was discovered
			- Also known as azimuth or yaw
		- Complies to vehicle specific motion constraints
		- Ensures resulting movements are smooth
	Methods
		- Geometric Methods
			- Utitlize look ahead distance to measure error ahead of vehicle
				- Extend from simple circular arc calculations to much more complex geometric theorems, such as vector pursuit
			- Relatively simple to implement, more robust to path curvatures, work better for lower speed driving
		- Model Based Methods
			- Higher tracking accuracy at higher speeds where the effects of non-linear vehicle dynamics have to be taken into account
			- Use either first order (kinematic) or second order (dynamic) model of vehicle
			- Perform well for highway driving, continuous paths
			- Not robust to disturbances and large lateral offsets
			- Cut corners, rapidly accelerates and decelerates, and pursures paths with large curvature
	Pure Pursuit Geometric Method
		- pursures a point along a path that is located a certain distance away from vehicle's current pos
		- relatively simple to implement, robust to disturbances and large lateral error
		- waypoints input (not smooth curves)
		- less susceptible to discretization related issues
		- suffers from corner cutting and steady state error problems if lookahead distance selected is not appropriate, especially at higher speed, when lookahead distance increases
	Stanley Geometric Method
		- considers cross track error of the vehicle to the path, as measured from the front axle of the vehicle, as well as the heading error of the vehicle with respect to the path
		- Exponentially converges to 0 cross track error
		- Better tracking results as it does not cut corners
		- Better at high speed driving
		- However, not robust to disturbances and higher tendency for oscillation
		- Requries continuous curvature path rather than waypoints
			- Making it susceptible to descritization related issues
	Improved Pure Pursuit
		- Considers the heading orientation of the pursuit point
		- Reduces corner cutting previously encountered
		- Kinematic Car Model is used (Ackermann steering geometry)
			- Assumes each vehicle tire moves along a circular arc with common center of rotation
		- Assumption that there is zero lateral motion on tires may have negative impact as velocity increases and path curvature varies
		- Some modern approaches use dynamic models of the vehicle to design steering controllers
		- However, dynamics of the car can be complicated. Dynamic controllers can bring great computation cost. Many tradeoffs that ultimately make dynamic kinematic models undesirable.
# Vehicle Model
		![[Pasted image 20220826103543.png]]
		- velocity is in xb direction
		- yaw (theta) and yaw rate (theta mark) are related to front wheel steering angle (partial d) by equation: ![[Pasted image 20220826103854.png]]
		- L is vehicle wheelbase (distance between the  front axle and the rear axle of the vehicle)
		![[Pasted image 20220826104131.png]]
	- Relation between steering angle and curvature radius of path can be approximated as: ![[Pasted image 20220826104444.png]]
# Pure Pursuit Path Tracking
![[Pasted image 20220829191428.png]]
## Path Representation
- To prevent constricting the way the paths are generated, the paths shall be represented in a piece wise linear way
- Discrete nodes are contained in an ordered list and they are tracked sequentially
## Algorithm
- Single tunable parameter, lookahead distance
- Defines a virtual circular arc that connects the anchor point (the rear axle) to the tracked point along the path that is located lookahead away from the anchor point.
- Anchor point can be not rear axle, but in paper, it is assumed to be
- Virtual arc is constrained to be tangential to the velocity vector at the origin of the body fixed frame and to pass through the tracked point along the path. 
- Geometrically, using the law of sines, the radius of the curvature of the arc R can be computed as:
	![[Pasted image 20220829202510.png]]
- R is the turning radius, and η is the lookahead heading. Curvature κ of the arc is defined as ![[Pasted image 20220829202701.png]], κ is defined as 1/R
![[Pasted image 20220829203336.png]]
- So steering angle can be computed using kinematic model:
![[Pasted image 20220829203852.png]]
- Lookahead distance is often formulated a function of longitudinal velocity (saturated at certain max and min values)
![[Pasted image 20220830223420.png]]

