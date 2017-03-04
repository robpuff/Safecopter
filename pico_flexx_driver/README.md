# PMD CamBoard pico flexx Driver

## Maintainer

- [Thiemo Wiedemeyer](https://ai.uni-bremen.de/team/thiemo_wiedemeyer) <<wiedemeyer@cs.uni-bremen.de>>, [Institute for Artificial Intelligence](http://ai.uni-bremen.de/), University of Bremen

## Table of contents
- [Description](#description)
- [Dependencies](#dependencies)
- [Install](#install)
- [Usage](#usage)

## Description

This package is a ROS interface to the [CamBoard pico flexx](http://www.pmdtec.com/picoflexx/) from pmd.
The included node publishes the point cloud, camera info, depth, ir and noise images on ROS topics.
It supports dynamic reconfigure and nodelets with zero copy transfers.
A launch file with static TF publisher, nodelet manager and machine tag support is included.

## Dependencies

- ROS Indigo (or newer should also work)
- [libroyale](http://www.pmdtec.com/picoflexx/) (1.0.5.40 or newer)

## Install

1. Install the ROS. [Instructions for Ubuntu 14.04](http://wiki.ros.org/indigo/Installation/Ubuntu)
2. [Setup your ROS environment](http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment)
3. Download the Royale SDK from http://www.pmdtec.com/picoflexx/ and extract it
4. Extract the linux 64 bit archive from the extracted SDK.
5. Install the headers and library from the SDK using the provided [script](install_libroyale.sh)
   ```
cd <catkin_ws>/src/pico_flexx_driver
sudo ./install_libroyale.sh <path to extracted linux 64 bit archive>
```
6. Run `catkin_make`
7. Plug in the CamBoard pico flexx device
8. Run `roslaunch pico_flexx_driver pico_flexx_driver.launch publish_tf:=true`
9. Start `rosrun rviz rviz`, set the `Fixed frame` to `pico_flexx_link` and add a `PointCloud2` and select `/pico_flexx/points`

## Usage

To start the pico flexx driver, please use the provided launch file: `roslaunch pico_flexx_driver pico_flexx_driver.launch`

#### Parameters

The launch file has the following paramters:

- `base_name` (default="pico_flexx"):
  Name of the node. All topics will be advertised under this name.
- `sensor` (default=""):
  ID of the sensor that should be used. IDs of all connected devices are listed on startup.
- `operation_mode` (default="0"):
  ID of the operation mode A list of supported modes is listed on startup.
- `automatic_exposure` (default="true"):
  Enable or disable automatic expusure.
- `exposure_time` (default="1000"):
  Exposure time. Only for manual exposure.
- `max_noise` (default="0.07"):
  Maximum allowed noise. Data with higher noise will be filtered out.
- `range_factor` (default="2.0"):
  Range of the 16-Bit mono image which should be mapped to the 0-255 range of the 8-Bit mono image. The resulting range is `range_factor` times the standard deviation arround mean.
- `queue_size` (default="5"):
  Queue size for publisher.
- `publish_tf` (default="false"):
  Publish a static TF transform for the optical frame of the camera.
- `base_name_tf` (default="pico_flexx"):
  Base name of the tf frames.
- `machine` (default="localhost"):
  Machine on with the nodes should run.
- `define_machine` (default="true"):
  Whether the machine for localhost should be defined our not. Disable this if the launch file is included somewhere where machines are already defined.
- `nodelet_manager` (default="pico_flexx"):
  Name of the nodelet manager.
- `start_manager` (default="true"/>
  Whether to start a nodelet manager our not. Disable this if a different nodelet manager should be used.

#### Dynamic reconfigure

Some parameters can be reconfigured during runtime, for example with `rosrun rqt_reconfigure rqt_reconfigure`. The reconfigureable parameters are:
- `operation_mode`:
  Choose from a list of operation modes.
- `exposure_mode`:
  `AUTOMATIC` or `MANUAL`.
- `exposure_time`:
  Exposure time. Only for manual exposure.
- `max_noise`:
  Maximum allowed noise. Data with higher noise will be filtered out.
- `range_factor`:
  Range of the 16-Bit mono image which should be mapped to the 0-255 range of the 8-Bit mono image. The resulting range is `range_factor` times the standard deviation arround mean.

#### Topics

##### `/pico_flexx/camera_info`
Bandwidth: 0.37 KB per message (@5 Hz: ~2 KB/s, @45 Hz: ~ 17 KB/s)

This topic publishes the camera intrinsic parameters.

##### `/pico_flexx/image_depth`
Bandwidth: 153.28 KB per message (@5 Hz: ~766 KB/s, @45 Hz: ~ 6897 KB/s)

This is the undistorted depth image. It is a 32-Bit float image where each pixel is a distance measured in meters.

##### `/pico_flexx/image_mono16`
Bandwidth: 76.67 KB per message (@5 Hz: ~383 KB/s, @45 Hz: ~ 3450 KB/s)

This is the undistorted IR image. It is a 16-Bit image where each pixel is an intensity measurement.

##### `/pico_flexx/image_mono8`
Bandwidth: 38.37 KB per message (@5 Hz: ~192 KB/s, @45 Hz: ~ 1727 KB/s)

This is the undistorted IR image. It is a 8-Bit image where each pixel is an intensity measurement.

##### `/pico_flexx/image_noise`
Bandwidth: 153.28 KB per message (@5 Hz: ~766 KB/s, @45 Hz: ~ 6897 KB/s)

This is the undistorted noise image. It is a 32-Bit float image where each pixel is a noise value of the corresponding depth pixel measured in meters.

##### `/pico_flexx/points`
Bandwidth: 770 KB per message (@5 Hz: ~3850 KB/s, @45 Hz: ~ 34650 KB/s)

This is the point cloud created by the sensor. It contains 6 fields in the following order: X, Y, Z, Noise (float), Intensity (16-Bit), Gray (8-Bit).
The X, Y, Z coordinates are undistorted. The point cloud is organized, so that the each point belongs to the pixel with the same index in one of the other images.
