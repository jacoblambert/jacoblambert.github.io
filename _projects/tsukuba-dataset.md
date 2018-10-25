---
title: 'Tsukuba Challenge Dynamic Object Tracks Dataset'
date: 2018-03-30
permalink: /projects/tsukuba-dataset/
tags:
  - dataset
  - academia
---
<h2>Tsukuba Challenge Dynamic Object Tracks Dataset</h2>

The MAD team presents the Tsukuba Challenge Dynamic Object Tracks Dataset which you can access <a href="https://github.com/MAVRG/meidai-data-tools/tree/master/tsukuba2017_jrm">on Github</a>

We collected 15 rosbags runs of the 2017 <a href="http://www.tsukubachallenge.jp/">Tsukuba Challenge</a> each of which covers the a distance of over 2 kilometers. The following is a top view of the course, where our trajectory is in red and approximate traversable area is in green: <img src="http://jacoblambert.github.io/images/tsukuba/tsukubaearthgood2.png" alt="tsukubaearthgood2.png" width="1637" height="867" />

<img src="http://jacoblambert.github.io/images/tsukuba/minbotsensors.png" alt="minbotsensors" align="right" width="191" height="300" />
We use the MinBot to collect this data, which you can see below. It is a manually operated data collection platform, equipped with many sensors: Velodyne HDL-32 3D lidar, Hokuyo UTM-30LX 2D lidar, PointGrey Flea3 monocular camera, Xsens MTi-300 intertial motion unit (IMU) as well as wheel encoders.

Though we logged information for all sensors, note that for this research we use the Velodyne HDL-32 3D lidar alongside wheel odometry for localization, and only 3D lidar for dynamic object detection and tracking.

We use <a href="https://github.com/CPFL/Autoware">Autoware</a>'s NDT Mapping and Localization tool to create a 3D Lidar Map of the environment which we use, alongside odometry, to localize. Then, using multiple runs of the environment, we use <a href="https://octomap.github.io/">Octomap</a> to create a 3D occupancy grid of the environment's static background. The final map of the environment can be seen below.

<img src="http://jacoblambert.github.io/images/tsukuba/octomap.png" alt="octomap" width="1211" height="705" />

With localization capabilities and a static background map, we can now perform the dynamic object detection and tracking from the Lidar data:

<img src="http://jacoblambert.github.io/images/tsukuba/block_diagram.png" alt="pflow" width="3529" height="5257" />

Points remaining after ground and background removal are then clustered using PCL's <a href="http://www.pointclouds.org/documentation/tutorials/cluster_extraction.php">Euclidean Clustering</a>. This was often insufficient as pedestrians naturally move in clusters so several people are often grouped as one. To solve this issue we performed multiple maxima (heads) iterative sub-clustering. This is an important step as illustrated below. A group of 6 people block the road. Due to the point of view of the 3D Lidar, three people on the right are grouped into one cluster, one of which is heavily occluded. After sub-clustering, we have two clear people, with the third being rejected since cluster size is not big enough to truly determine whether this is a person.

<img src="http://jacoblambert.github.io/images/tsukuba/people.png" alt="pflow" width="3529" height="5257" />

The occluded person may eventually come into view and be added into our list of tracked objects. At each lidar frame, we look for new objects, perform data association and track existing objects using a particle filter. We also interpolate the trajectories of objects using b-spline, both to predict future position and to smooth out existing trajectories which can be noisy due to lidar noise.

Also due to Lidar noise or poor localization, we often pick up background as dynamic objects for few milliseconds. This is especially problematic in areas like the Tsukuba Tent Area, which has a lot of shifting background. We can however filter out these false positives fairly easily by checking some statistics of the trajectories. We filter out trajectories with unnatural speed or rotational velocity, as well as objects that were tracked for very short distance of amount of time. Below, you can see all good trajectories in green, and all filtered our trajectories in red:

<img src="http://jacoblambert.github.io/images/tsukuba/red_green_zoom.png" alt="red_green_zoom.png" width="3710" height="2044" />After filtering, we have a sanitized dataset of dynamic objects with the following information:
<ul>
	<li><strong>Map of the Tsukuba Environment</strong>: 3D point cloud map and 2D occupancy grid map of the Tsukuba challenge environment.</li>
	<li><strong>Robot Trajectory</strong>: position of the robot inside the map at each timestamp.
Dynamic Object Locations: 2D and 3D locations of dynamic objects, with object ID and timestamp, localized in a global coordinate frame.</li>
	<li><strong>2D Smoothed Trajectories</strong>: b-spline-derived smooth trajectories.</li>
	<li><strong>Velocity and Heading</strong>: derived velocity and heading from the smooth trajectory for each dynamic objects.</li>
	<li><strong>Bounding Box</strong>: 3D bounding boxes and centroid location of each object.</li>
	<li><strong>Object Point Clouds</strong>: 3D points inside each objects bounding box.</li>
</ul>
The data is currently available on <a href="https://goo.gl/k2pGHE">Google Drive</a>.

<h2>Data analysis tools</h2>
In this section I'll go over the <a href="https://github.com/jacoblambert/meidai-data-tools">Meidai Data Tools</a> module for the 2017 Tsukuba Challenge Dataset.

This python module includes two packages:
<ul>
	<li>dict_tools: some tools demonstrating how to load the data from the dictionary and how the dictionary was created from the raw text files.</li>
	<li>vis_tools: tools at your convenience to visualize the data.</li>
</ul>
First, you need to install the module, simply:
<code>python setup.py install</code>

The module has a few requirements that are not included by default in python:
<ul>
	<li>numpy</li>
	<li>matplotlib</li>
	<li>seaborn</li>
</ul>
<h2>dict_tools</h2>
The dictionary tools module gives you functions and examples to load the Tsukuba Challenge data which is store in python dictionaries inside of pickle files. <code> loader.py </code> contains all the information you need to access the data inside the pickle '.pk1' files:

The pickle file holds a dictionary with 4 keys, each in turn holding a dictionary with the trajectory ID as key
<ul>
	<li><strong>'raw_tracks'</strong> : raw trajectory data</li>
	<li><strong>'tracks'</strong> : b-spline smoothed trajectories</li>
	<li><strong>'filtered'</strong> : b-spline trajectories filtered for false positives with derived heading, velocity and rotational velocity (see paper or create_dict for filtering details)</li>
	<li><strong>'pointclouds' </strong>: segmented pointclouds for each object</li>
</ul>
Some illustrative code snippets:
(or any other of the above 4 keys) returns all track IDs as keys in that dictionary.
<code>print dataset['raw_tracks'].keys()</code> is also a dictionary.
<code>print dataset['pointclouds'][track_ID].keys()</code> returns all the timestamps at which the dataset was observed, as keys to that dictionary.

The data is stored in numpy ndarrays of dtype=float32:
<ul>
	<li>map_data is a Nx3 matrix of points, centroids for the voxel grid</li>
	<li>dataset['raw_tracks'][track_ID] : N x 7
[timestamp, position_x, position_y, position_z, orientation_w, variance_x, variance_y]
approximating the 2D particle filter distrubution as a gaussian, we get the above variance</li>
	<li>dataset['tracks'][trackID] : N x 3
[obstime, positionx, positiony]</li>
	<li>dataset['filtered'] : N x 7, [timestamp. position_x, position_y, heading, velocity_x, velocity_y, rotation_w]
heading is from +x-axis, ccw [-pi, pi]</li>
	<li>dataset['pointclouds'][trackID][timestamp] : K x 3, [x, y, z]</li>
</ul>
You can run the loader through command line:
<code>python loader.py path/to/dataset.pk1 path/to/map.txt</code>
but you can also run it in your favorite IDE and change the <code>dataset_file</code> and <code>map_file</code>
<h2>vis_tools</h2>
These scripts show how you can plot various aspects of the data. All run the same way as the loader.
<ul>
	<li><code>plot_tracks.py</code>: plots the track as simple lines with the map.</li>
</ul>
<img src="http://jacoblambert.github.io/images/tsukuba/tracks.png" alt="tracks.png" width="2318" height="1242" />
<ul>
	<li><code>plot_velocity.py</code>: plots the track colored by average velocity. You can set the minimum and maximum velocity of the colormap inside the script.</li>
</ul>
<img src="http://jacoblambert.github.io/images/tsukuba/vel.png" width="2318" height="1242" />
<ul>
	<li><code>plot_headings.py</code>: plots the trajectories by current settings, changing color every few timesteps, controlled by the <code>stride</code> parameter.</li>
</ul>
<img src="http://jacoblambert.github.io/images/tsukuba/heads.png" alt="heads.png" width="2318" height="1242" />
<ul>
	<li><code>plot_pointclouds.py</code>: plots segmented pointclouds, for each track ID then each timestep. The script plots the next timestep when you close the window. If the color of the points change, the track_ID changed.</li>
	<li><code>plot_pointcloud_seq.py</code>: plots segmented pointclouds, for each track ID then each timestep like the previous script, but keeps around a few (up to 4) timesteps of point with progressively stronger transparent. The current timestep has no alpha and black border.</li>
</ul>