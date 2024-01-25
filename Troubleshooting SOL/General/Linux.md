Sources are also repositories (means same thing)

Give folder ownership
	sudo chown -R user path
	OR
	sudo chmod -R 777 path

Fix broken apt
	sudo apt --fix-broken install

Dpkg overwrite through apt install
	sudo apt-get -o Dpkg::Options::="--force-overwrite" --fix-broken install

Repo Removal
	Remove repositorys from sources.list and sources.list.d (/etc/apt/)
	Remove apt-key list (apt-key del <key>)

Abraunegg Onedrive client
	OpenSuse repo is added to apt repos
	Apt installs and manages onedrive
	To delete
		Remove and purge onedrive and delete repos

GNURadio 
	Repository added
	Managed by apt

ROS2 Humble Jammy 22.04
	 Install
		https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html
		ROS2 repo is added to apt repos
		Apt installs and manages ros2
		Apt installs and manages rosdep
		Applied nav2 from source
	.bashrc
		export ROS_DOMAIN_ID=55
		export ROS_LOCALHOST_ONLY=1
		source /opt/ros/humble/setup.bash
		alias sourceros=". install/setup.bash"
	Env
		mkdir -p ~/ros2_ws/src
		clone package
	Build
		cd to root ws
		colcon build --symlink-install
		sourceros
	 READY TO GO
Add Nav2
	Install
		cd to src
		clone package (https://github.com/ros-planning/navigation2.git --branch <ros2-distro>-devel)
		cd to root ws
		install all dependencies (rosdep install -y -r -q --from-paths src --ignore-src --rosdistro <ros2-distro>)
	Build
		colcon build --symlink-install
		--cmake-clean-cache
		--cmake-clean-first (rebuild)
		--cmake-force-configure
		--packages-skip nav2_system_tests

LOTUS Control Service ROS2 Workspace creation
sudo apt install ros-humble-desktop
clone jmogainz/vesc forked ros2-rebase-fixed
clone jmogainz/two_wheeled_robot forked
clone f1tenth/transport_drivers main
clone kumarrobotics/ublox ros2

Singularity
	sudo singularity build --writable ros2.sif ros2.def
	sudo singularity shell --writable ros2.sif
		
		
	





