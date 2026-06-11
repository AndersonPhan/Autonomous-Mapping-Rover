# Autonomous-Mapping-Rover

## Abstract

This report documents the development of an autonomous rover designed and built as part of a senior design project. The rover can map an unknown environment within an arena using a 2D LiDAR sensor and SLAM (Simultaneous Localization and Mapping) and autonomously navigate between two points using a saved map and a pathfinding algorithm, all coordinated through the ROS2 framework running on a Raspberry Pi 5. Additionally, a YOLOv5 machine learning model was trained and deployed onboard to enable real-time object detection through the rover's camera. The system is organized into five interconnected subsystems: Processing, Sensing, Mobility, Power, and Base Station, each contributing to the rover's fully autonomous pipeline. This report details the design, integration, verification, and operation of the final product, as well as key lessons learned throughout the development process

## Project Definition

The definition for this project was to create a rover that can map an arena using ROS2 libraries and a 2D Lidar to execute SLAM. The rover then should be able to autonomously move from point A to point B by using a saved map on rviz and a path finding algorithm.

## How it works

### Parts List

<p align="center">
  <img width="515" height="228" alt="image" src="https://github.com/user-attachments/assets/e87cc539-2cd3-4bc7-af34-e393ae83ba52" />
</p>
<p align="center">
  <em><b>Fig 1:</b> Parts Implemented on the rover</em>
</p>

### Rover Subsystem Diagram

As shown in Figure 2, the system is organized into five different interconnected subsystems. The first of these systems is the Processing Subsystem, which is the core of the rover. This subsystem is built around Raspberry Pi 5, which runs ROS2 and YOLOv5, handing the computing, path finding, and coordination of the other subsystems. The second subsystem is the Sensing Subsystem, which includes the 2D Lidar Sensor and Camera. This subsystem is responsible for gathering data for the processing unit in order to execute object detection, mapping, and navigation. The third subsystem is the Mobility Subsystem, which consists of the motor control unit (RCC Lite Controller) and the four motors and wheels. The wheels are mecanum wheels, which allow for omnidirectional movement. The fourth subsystem is the Power Subsystem, which provides power to the rover through a lithium battery and the PCB, which acts as a power distribution board. The fifth and final subsystem is the Base Station, which consists of a desktop/laptop running Rviz on Real VNC Viewer. The laptop allows for wireless monitoring and Rviz allows us to observe the Lidar data on a 2D map. Together these subsystems form an autonomous robot pipeline, from sensing the environment to visualizing and controlling the platform.

<p align="center">
  <img width="788" alt="System Diagram" src="https://github.com/user-attachments/assets/b74bac6e-d512-490a-86aa-072e194d2d8f">
</p>
<p align="center">
  <em><b>Fig 2:</b> Detailed diagram of subsystems and component connections.</em>
</p>

### Setup

**Step 1** — Arena Setup: Begin by arranging the physical arena, which consists of a bounded area with objects or walls that the rover will map. Place the rover inside the arena on a flat surface and ensure the LiDAR sensor and camera are unobstructed. Power on the rover using the lithium battery and confirm that the Raspberry Pi 5 has booted successfully.<br>

**Step 2** — Connect the Base Station: On a laptop, open RealVNC Viewer and connect wirelessly to the Raspberry Pi 5. This gives you remote access to the rover's desktop environment. Open a terminal window through the VNC session to begin entering ROS2 commands.<br>

**Step 3** — Mapping the Arena with SLAM: Launch the SLAM and LiDAR ROS2 nodes by entering the appropriate launch commands in the terminal. As the rover moves through the arena, the LiDAR continuously scans the surroundings and the SLAM algorithm builds a 2D occupancy map in real time, which is visualized in RViz on the Base Station laptop. At this stage, the rover can be manually driven around the arena using a controller, allowing the ability to see the map update live as the rover explores. Once the full arena has been mapped, the map is saved for use in autonomous navigation.<br>

**Step 4** — Loading the Saved Map and Autonomous Navigation: After the map is saved, the SLAM nodes are stopped and the navigation stack is launched using another ROS2 command. The saved map is loaded into RViz, and a target destination point is set by clicking directly on the map in RViz. The rover then uses its pathfinding algorithm to autonomously calculate a route and drive itself from its current location to the designated point without any human input.<br>

**Step 5** — Object Detection: Throughout the operation, the onboard YOLOv5 model runs continuously in the background, using the rover's camera to detect and classify objects in real time. Detection results are displayed on the Base Station, allowing for a camera view of what the rover is identifying as it moves through the arena. <br>

### Mapping

The 2D Lidar sensor is always scanning the environment and generates data based on the distance between the laser and the objects around it. It then feeds this raw data into an algorithm to build a 2D map, which can be visualized on Rviz on a laptop running Real VNC Viewer. This process can be seen in Figure 3.

To run SLAM and start the mapping process the following commands need to be runned:

| Command | Function |
|----------|----------|
| `ros2 launch slam slam.launch.py` | Launch SLAM so that the LiDAR can start generating data and rover movement is enabled. |
| `ros2 launch slam rviz2_rtabmap.launch` | Launch RViz2 so that the Raspberry Pi 5 translate LiDAR data into a 2D map that can be visualized. |
| `ros2 launch peripherals teleop_key_control.launch.py` | Launch Teleop Keys to control the rover using a keyboard from the base station. |

<table align="center">
  <tr>
    <td align="center">
      <img width="337" height="449" alt="image" src="https://github.com/user-attachments/assets/3be89ff5-108d-4a0b-ba06-a0c8fb2e5ad1" /><br>
      2D Map of arena visualized on rviz; the rainbow outlines represents what the LiDAR is currently scanning
    </td>
    <td align="center">
      <img width="337" height="449" alt="image" src="https://github.com/user-attachments/assets/6e5652a5-8b83-422c-94a9-ad0b6d95f01b" /><br>
      Physical Arena being mapped
    </td>
  </tr>
</table>
<p align="center">
  <b>Fig 3:</b> Arena Mapping
</p>

### Machine Learning

A Yolov5 Model was used to develop a computer vision-based object detection model using YOLOv5. The model identifies two key object classes: Wooden Block and Ramp.

#### Pipeline

<p align="center">
  <img width="883" height="272" alt="image" src="https://github.com/user-attachments/assets/894956b5-73a0-4c13-b2f7-6f12978070cc" />
</p>
<p align="center">
  <em><b>Fig 4:</b> Machine Learning Pipeline</em>
</p>

The dataset was created by extracting frames from recorded videos of the rover environment. These images were manually annotated on a platform called Roboflow to identify ramps and wooden blocks.

The model was then trained using Google Colab, which provided GPU acceleration for faster training.

During training, the model learns to:

1. Identify objects within an image.
2. Classify each object as either a ramp or a wooden block.
3. Generate bounding boxes around detected objects.

During execution the trained YOLOv5 model then processes incoming camera images and outputs:

- Object class labels
- Bounding box locations
- Detection confidence scores

#### Model Implementation
After the YOLOv5 model was trained in Google Colab, the resulting best.pt file containing the trained weights was exported and transferred to the rover's Raspberry Pi. The Raspberry Pi executes the object detection model in real time using a connected camera as the primary input source. he implementation consists of three main components:

- A camera for capturing live video of the rover's surroundings.
- The YOLOv5 object detection script running on the Raspberry Pi.
- The ROS visualization tool rqt, which displays the camera feeds and detection results.

As shown in the figure below, the left side displays the live camera feed captured by the rover. The right side displays the processed video stream at a reduced frame rate, allowing sufficient computational resources for the YOLOv5 model to perform object detection accurately. The model successfully identifies wooden blocks within the environment and places green bounding boxes around the detected objects.

<p align="center">
  <img width="537" height="300" alt="image" src="https://github.com/user-attachments/assets/109c116a-4cfe-4535-9dc3-985f93dac252" />
</p>
<p align="center">
  <em><b>Fig 5:</b> Live camera view on rover</em>
</p>

### Pathfinding

After a map is saved onto the Pi; the rover can then receive input commands to send it from point A to point B. This is done by using the 2D point estimation tool where the green arrow is the drection the rover will move in.

<p align="center">
<img width="608" height="386" alt="image" src="https://github.com/user-attachments/assets/856d6b01-75d3-494e-aa13-d1091289cfac" />
</p>
<p align="center">
  <em><b>Fig 6:</b> Pathfinding using saved map</em>
</p>

## Live Demo

[![Watch the demo](https://img.youtube.com/vi/5KnCUVMfur4/maxresdefault.jpg)](https://www.youtube.com/watch?v=5KnCUVMfur4)

## Conclusions

This senior design project resulted in the development of an autonomous rover that can map an unknown arena and navigate between two points. Through the use of ROS2, 2D Lidar, SLAM, and a YOLOv5 machine learning model, the team was able to create a fully functional system that demonstrates the real-world applications of autonomous navigation and object detection. Despite the overall challenges that were faced this project was able to provide me with value experience in embedded systems and robotics middleware for my future career.

## Safety and Ethical Considerations:

The rover was designed with certain safety and ethical considerations in mind.

**IEEE 315-1975, ISO 66469-1:2019**
Focuses on general safety requirements, for a system to comply with standards it must meet criteria’s during testing: no evidence of electrolyte leakage, nor fire or explosion, no rupture, and no electrical isolation, and designing circuits with correct symbols, and building a efficient circuit with correct components.

**IEEE 112-2017, ISO 13849-1**
Its primary goal is to create consistent user experience across different types of devices, making energy-saving power management features easier to understand and use. Devices should have consistent behavior and responsiveness within their substates

**IEEE 7000-2021, ISO/IEC 22989:2022**
An international standard that establishes a comprehensive framework of terminology and concepts for AI. This document describes concepts to support the development of other standards and facilitates communication among diverse parties.

**IEEE 1872-2015, ISO/IEC 27001**
Defines the most fundamental and general concepts in robotics and automation domain. Components and ontologies mentioned in the document are CORAX, POS. POS captures general concepts about position, orientation, and pose which are critical for spatial reasoning in robotics.

**IEEE 2846-2022, ISO/IEC TR 24368**
Establishes framework for the assumptions on automated driving systems. This standard aims to balance safety with natural driving behavior.

## References

[1] “Standards.” IEEE Standards Association, 12 Sept. 2023, standards.ieee.org/standard/.<br>

[2] “Popular Standards.” ISO, 31 Jan. 2024, www.iso.org/popular-standards.html.<br>

[3] HiWonder. “Mentorpi.” MentorPi - MentorPi v2.0 Documentation, 2025, wiki.hiwonder.com/projects/MentorPi/en/latest/.<br>

