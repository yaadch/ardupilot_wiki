.. _common-mavlink-mission-command-messages-mav_cmd:

================
Mission Commands
================

This article describes the mission commands that are supported by Copter, Plane, Sub and Rover when switched into Auto mode.
[site wiki="copter"]
A simpler list just for :ref:`Copter can be found here <copter:mission-command-list>`
[/site]

Overview
========

The MAVLink protocol defines a large number of
`MAV_CMD <https://github.com/ArduPilot/mavlink/blob/master/message_definitions/v1.0/common.xml#L1008>`__
waypoint command types (sent in a ``MAVLink_mission_item_message``).
ArduPilot implements handling for the subset of these commands and
command parameters that are *most relevant* and meaningful for each of
the vehicles. Unsupported commands that are sent to a particular
autopilot will simply be dropped.

This article lists and describes the commands and command-parameters
that are supported on each of the vehicle types. Any parameter that is
"grey" is not supported by the autopilot and will be ignored. They are
still documented to make it clear which properties are supported by
the `MAV_CMD protocol <https://github.com/ArduPilot/mavlink/blob/master/message_definitions/v1.0/common.xml#L1008>`__
are not implemented by the vehicle.

Some commands and command parameters are not implemented because they
are not relevant for particular vehicle types (for example
"MAV_CMD_NAV_TAKEOFF" command makes sense for Plane and Copter but
not Rover and the pitch parameter only makes sense for Plane). There
are also some potentially useful command parameters that are not handled
because there is a limit to the message size, and a decision has been
made to prioritize some parameters over others.

.. note::

   There is additional information about the supported commands on
   Copter (from a Mission Planner perspective) in the :ref:`Copter Mission Command List <copter:mission-command-list>`.

Types of commands
-----------------

There are several different types of commands that can be used within
missions:

-  Navigation commands are used to control the movement of the vehicle,
   including takeoff, moving to and around waypoints, changing altitude,
   and landing.
-  DO commands are for auxiliary functions and do not affect the
   vehicle’s position (for example, setting the camera trigger distance,
   or setting a servo value).
-  Condition commands are used to delay DO commands until some condition
   is met, for example, the UAV reaches a certain altitude or distance
   from the waypoint.

During a mission at most one “Navigation” command and one “Do” or
"Condition" command can be running at one time. A typical mission might
set a waypoint (NAV command), add a CONDITION command that doesn't
complete until a certain distance from the destination
(:ref:`MAV_CMD_CONDITION_DISTANCE <mav_cmd_condition_distance>`), and
then add a number of DO commands that are executed sequentially (for
example
:ref:`MAV_CMD_DO_SET_CAM_TRIGG_DIST <mav_cmd_do_set_cam_trigg_dist>`
to take pictures at regular intervals) when the condition completes.

.. note::

   CONDITION and DO commands are associated with the preceding NAV
   command: if the UAV reaches the next waypoint before these commands are
   executed, the next NAV command is loaded and they will be
   skipped.


.. _common-mavlink-mission-command-messages-mav_cmd_navigation_commands_frames:

Frames of reference
-------------------

Many of the commands (in particular the :ref:`NAV\_ commands <common-mavlink-mission-command-messages-mav_cmd_navigation_commands>`) include position/location
information. The information is provided relative to a particular "frame
of reference", which is specified in the message's :ref:`common-mavlink-mission-command-messages-mav_cmd_navigation_commands_frames` field. Copter and Rover Mission use :ref:`MAV_CMD_DO_SET_HOME <mav_cmd_do_set_home>` command to set the
"home position" in the global coordinate frame (MAV_FRAME_GLOBAL),
`WGS84 coordinate system <https://en.wikipedia.org/wiki/World_Geodetic_System>`__, where
the altitude is relative to mean sea level. All other commands use the
MAV_FRAME_GLOBAL_RELATIVE_ALT frame, which uses the same latitude
and longitude, but sets altitude as relative to the *home position*
(home altitude = 0).

Plane commands can additionally use MAV_FRAME_GLOBAL_TERRAIN_ALT
frame of reference. This again has the same WGS84 frame of reference for
latitude/longitude, but specifies altitude relative to ground height (as
defined in a terrain database).

.. note::

   The other frame types are defined in the MAVLink protocol (see
   `MAV_FRAME <https://github.com/ArduPilot/mavlink/blob/master/message_definitions/v1.0/common.xml#L795>`__)
   are not supported for mission commands.

How accurate is the information?
--------------------------------

If a command or parameter is marked as supported then it is likely (but
not guaranteed) that it will behave as indicated. If a command or
parameter is not listed (or marked as not supported) then it is
extremely likely that it is not supported on ArduPilot.

The reason for this is that the information was predominantly inferred
by inspecting the command handlers for messages:

-  The switch statement in `AP_Mission::mavlink_to_mission_cmd <https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Mission/AP_Mission.cpp#L466>`__
   was inspected to determine which commands are handled by *all*
   vehicle platforms, and which parameters from the message are stored.
-  The command handler switch for each vehicle type
   (`Plane <https://github.com/ArduPilot/ardupilot/blob/master/ArduPlane/commands_logic.cpp#L33>`__,
   `Copter <https://github.com/ArduPilot/ardupilot/blob/master/ArduCopter/commands_logic.cpp#L49>`__,
   `Rover <https://github.com/ArduPilot/ardupilot/blob/master/Rover/commands_logic.cpp#L25>`__)
   tells us which commands are likely to be supported in each vehicle
   and which parameters are passed to the handler.

The above checks give a very accurate picture of what commands and
parameters are not supported. They give a fairly accurate picture of
what commands/parameters are *likely to be supported*. However, this
indication is not guaranteed to be accurate because a command handler
could just throw away all the information (and we have not fully checked
all of these).

In addition to the above checks, we have also merged information from
the :ref:`Copter Mission Command List <copter:mission-command-list>`.

How to interpret the command parameters
---------------------------------------

The parameters for each command are listed in a table. The parameters
that are "greyed out" are not supported. The command field column (param
name) uses "bold" text to indicate those parameters that are defined in
the protocol (normal text is used for "empty" parameters).

This allows users/developers to see both what is supported, and what
protocol fields are not supported in ArduPilot.

Using this information with a GCS
---------------------------------

*Mission Planner* (MP) exposes the full subset of commands and
parameters supported by ArduPilot, filtered to display just those
relevant to the currently connected vehicle. Mapping the MP commands to
this documentation is easy, because it simply names commands using a
cut-down version of the full command name (e.g. ``DO_SET_SERVO`` rather
than the full command name: ``MAV_CMD_DO_SET_SERVO``). In addition, this
document conveniently lists the column label used by Mission Planner
alongside each of the parameters.

Other GCSs (APM Planner 2, Tower etc.) may support some other subset of
commands/parameters and use alternative names/labels for them. In most
cases, the mapping should be obvious.

[site wiki="copter"]
Commands supported by Copter
============================

This list of commands was inferred from the command handler in
`/ArduCopter/mode_auto.cpp <https://github.com/ArduPilot/ardupilot/blob/master/ArduCopter/mode_auto.cpp#L388>`__. 

- :ref:`MAV_CMD_NAV_WAYPOINT <mav_cmd_nav_waypoint>`
- :ref:`MAV_CMD_NAV_RETURN_TO_LAUNCH <mav_cmd_nav_return_to_launch>`
- :ref:`MAV_CMD_NAV_TAKEOFF <mav_cmd_nav_takeoff>`
- :ref:`MAV_CMD_NAV_LAND <mav_cmd_nav_land>`
- :ref:`MAV_CMD_NAV_LOITER_UNLIM <mav_cmd_nav_loiter_unlim>`
- :ref:`MAV_CMD_NAV_LOITER_TURNS <mav_cmd_nav_loiter_turns>`
- :ref:`MAV_CMD_NAV_LOITER_TIME <mav_cmd_nav_loiter_time>`
- :ref:`MAV_CMD_NAV_SPLINE_WAYPOINT <mav_cmd_nav_spline_waypoint>`
- :ref:`MAV_CMD_NAV_GUIDED_ENABLE <mav_cmd_nav_guided_enable>` (NAV_GUIDED only)
- :ref:`MAV_CMD_NAV_DELAY <mav_cmd_nav_delay>`
- :ref:`MAV_CMD_NAV_PAYLOAD_PLACE <mav_cmd_nav_payload_place>`
- :ref:`MAV_CMD_DO_JUMP <mav_cmd_do_jump>`
- :ref:`MAV_CMD_JUMP_TAG<mav_cmd_jump_tag>`
- :ref:`MAV_CMD_DO_JUMP_TAG <mav_cmd_do_jump_tag>`
- :ref:`MAV_CMD_MISSION_START <mav_cmd_mission_start>`
- :ref:`MAV_CMD_COMPONENT_ARM_DISARM <mav_cmd_component_arm_disarm>`
- :ref:`MAV_CMD_CONDITION_DELAY <mav_cmd_condition_delay>`
- :ref:`MAV_CMD_CONDITION_DISTANCE <mav_cmd_condition_distance>`
- :ref:`MAV_CMD_CONDITION_YAW <mav_cmd_condition_yaw>`
- :ref:`MAV_CMD_DO_AUX_FUNCTION <mav_cmd_do_aux_function>`
- :ref:`MAV_CMD_DO_CHANGE_SPEED <mav_cmd_do_change_speed>`
- :ref:`MAV_CMD_DO_SET_HOME <mav_cmd_do_set_home>`
- :ref:`MAV_CMD_DO_SET_SERVO <mav_cmd_do_set_servo>`
- :ref:`MAV_CMD_DO_SET_RELAY <mav_cmd_do_set_relay>`
- :ref:`MAV_CMD_DO_REPEAT_SERVO <mav_cmd_do_repeat_servo>`
- :ref:`MAV_CMD_DO_REPEAT_RELAY <mav_cmd_do_repeat_relay>`
- :ref:`MAV_CMD_DO_DIGICAM_CONFIGURE <mav_cmd_do_digicam_configure>` (Camera enabled only)
- :ref:`MAV_CMD_DO_DIGICAM_CONTROL <mav_cmd_do_digicam_control>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_CAM_TRIGG_DIST <mav_cmd_do_set_cam_trigg_dist>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_ROI <mav_cmd_do_set_roi>`
- :ref:`MAV_CMD_DO_MOUNT_CONTROL <mav_cmd_do_mount_control>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_GIMBAL_MANAGER_PITCHYAW <mav_cmd_do_gimbal_manager_pitchyaw>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_PARACHUTE <mav_cmd_do_parachute>` (Parachute enabled only)
- :ref:`MAV_CMD_DO_GRIPPER <mav_cmd_do_gripper>`
- :ref:`MAV_CMD_DO_GUIDED_LIMITS <mav_cmd_do_guided_limits>` (NAV_GUIDED only)
- :ref:`MAV_CMD_DO_SET_RESUME_REPEAT_DIST <mav_cmd_do_set_resume_repeat_dist>`
- :ref:`MAV_CMD_DO_FENCE_ENABLE <mav_cmd_do_fence_enable>`
- :ref:`MAV_CMD_DO_WINCH<mav_cmd_do_winch>`
- :ref:`MAV_CMD_STORAGE_FORMAT <mav_cmd_storage_format>`
[/site]

[site wiki="sub"]
Commands supported by Sub
=========================

This list of commands was inferred from the command handler in
`/ArduSub/commands_logic.cpp <https://github.com/ArduPilot/ardupilot/blob/master/ArduSub/commands_logic.cpp#L7>`__. 

- :ref:`MAV_CMD_NAV_WAYPOINT <mav_cmd_nav_waypoint>`
- :ref:`MAV_CMD_NAV_RETURN_TO_LAUNCH <mav_cmd_nav_return_to_launch>`
- :ref:`MAV_CMD_NAV_LAND <mav_cmd_nav_land>`
- :ref:`MAV_CMD_NAV_LOITER_UNLIM <mav_cmd_nav_loiter_unlim>`
- :ref:`MAV_CMD_NAV_LOITER_TURNS <mav_cmd_nav_loiter_turns>`
- :ref:`MAV_CMD_NAV_LOITER_TIME <mav_cmd_nav_loiter_time>`
- :ref:`MAV_CMD_NAV_GUIDED_ENABLE <mav_cmd_nav_guided_enable>` (NAV_GUIDED only)
- :ref:`MAV_CMD_NAV_DELAY <mav_cmd_nav_delay>`
- :ref:`MAV_CMD_DO_JUMP <mav_cmd_do_jump>`
- :ref:`MAV_CMD_JUMP_TAG <mav_cmd_jump_tag>`
- :ref:`MAV_CMD_DO_JUMP_TAG <mav_cmd_do_jump_tag>`
- :ref:`MAV_CMD_MISSION_START <mav_cmd_mission_start>`
- :ref:`MAV_CMD_COMPONENT_ARM_DISARM <mav_cmd_component_arm_disarm>`
- :ref:`MAV_CMD_CONDITION_DELAY <mav_cmd_condition_delay>`
- :ref:`MAV_CMD_CONDITION_DISTANCE <mav_cmd_condition_distance>`
- :ref:`MAV_CMD_CONDITION_YAW <mav_cmd_condition_yaw>`
- :ref:`MAV_CMD_DO_AUX_FUNCTION <mav_cmd_do_aux_function>`
- :ref:`MAV_CMD_DO_CHANGE_SPEED <mav_cmd_do_change_speed>`
- :ref:`MAV_CMD_DO_SET_HOME <mav_cmd_do_set_home>`
- :ref:`MAV_CMD_DO_SET_SERVO <mav_cmd_do_set_servo>`
- :ref:`MAV_CMD_DO_SET_RELAY <mav_cmd_do_set_relay>`
- :ref:`MAV_CMD_DO_REPEAT_SERVO <mav_cmd_do_repeat_servo>`
- :ref:`MAV_CMD_DO_REPEAT_RELAY <mav_cmd_do_repeat_relay>`
- :ref:`MAV_CMD_DO_DIGICAM_CONFIGURE <mav_cmd_do_digicam_configure>` (Camera enabled only)
- :ref:`MAV_CMD_DO_DIGICAM_CONTROL <mav_cmd_do_digicam_control>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_CAM_TRIGG_DIST <mav_cmd_do_set_cam_trigg_dist>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_ROI <mav_cmd_do_set_roi>`
- :ref:`MAV_CMD_DO_MOUNT_CONTROL <mav_cmd_do_mount_control>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_GIMBAL_MANAGER_PITCHYAW <mav_cmd_do_gimbal_manager_pitchyaw>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_PARACHUTE <mav_cmd_do_parachute>` (Parachute enabled only)
- :ref:`MAV_CMD_DO_GRIPPER <mav_cmd_do_gripper>`
- :ref:`MAV_CMD_DO_GUIDED_LIMITS <mav_cmd_do_guided_limits>` (NAV_GUIDED only)
- :ref:`MAV_CMD_DO_SET_RESUME_REPEAT_DIST <mav_cmd_do_set_resume_repeat_dist>`
- :ref:`MAV_CMD_DO_FENCE_ENABLE <mav_cmd_do_fence_enable>`
- :ref:`MAV_CMD_DO_WINCH <mav_cmd_do_winch>`
- :ref:`MAV_CMD_STORAGE_FORMAT <mav_cmd_storage_format>`
[/site]

[site wiki="plane"]
Commands supported by Plane
===========================

This list of commands was inferred from the command handler in
`/ArduPlane/commands_logic.cpp <https://github.com/ArduPilot/ardupilot/blob/master/ArduPlane/commands_logic.cpp#L33>`__. 

- :ref:`MAV_CMD_NAV_WAYPOINT <mav_cmd_nav_waypoint>`
- :ref:`MAV_CMD_NAV_RETURN_TO_LAUNCH <mav_cmd_nav_return_to_launch>`
- :ref:`MAV_CMD_NAV_TAKEOFF <mav_cmd_nav_takeoff>`
- :ref:`MAV_CMD_NAV_LAND <mav_cmd_nav_land>`
- :ref:`MAV_CMD_NAV_LOITER_UNLIM <mav_cmd_nav_loiter_unlim>`
- :ref:`MAV_CMD_NAV_LOITER_TURNS <mav_cmd_nav_loiter_turns>`
- :ref:`MAV_CMD_NAV_LOITER_TIME <mav_cmd_nav_loiter_time>`
- :ref:`MAV_CMD_NAV_ALTITUDE_WAIT <mav_cmd_nav_altitude_wait>`
- :ref:`MAV_CMD_NAV_LOITER_TO_ALT <mav_cmd_nav_loiter_to_alt>`
- :ref:`MAV_CMD_NAV_CONTINUE_AND_CHANGE_ALT <mav_cmd_nav_continue_and_change_alt>`
- :ref:`MAV_CMD_NAV_VTOL_TAKEOFF <mav_cmd_nav_vtol_takeoff>`
- :ref:`MAV_CMD_NAV_VTOL_LAND <mav_cmd_nav_vtol_land>`
- :ref:`MAV_CMD_NAV_DELAY <mav_cmd_nav_delay>`
- :ref:`MAV_CMD_NAV_PAYLOAD_PLACE <mav_cmd_nav_payload_place>`
- :ref:`MAV_CMD_CONDITION_DELAY <mav_cmd_condition_delay>`
- :ref:`MAV_CMD_CONDITION_DISTANCE <mav_cmd_condition_distance>`
- :ref:`MAV_CMD_DO_AUX_FUNCTION<mav_cmd_do_aux_function>`
- :ref:`MAV_CMD_DO_CHANGE_SPEED <mav_cmd_do_change_speed>`
- :ref:`MAV_CMD_DO_ENGINE_CONTROL <mav_cmd_do_engine_control>`
- :ref:`MAV_CMD_DO_VTOL_TRANSITION <mav_cmd_do_vtol_transition>`
- :ref:`MAV_CMD_DO_SET_HOME <mav_cmd_do_set_home>`
- :ref:`MAV_CMD_DO_SET_SERVO <mav_cmd_do_set_servo>`
- :ref:`MAV_CMD_DO_SET_RELAY <mav_cmd_do_set_relay>`
- :ref:`MAV_CMD_DO_REPEAT_SERVO <mav_cmd_do_repeat_servo>`
- :ref:`MAV_CMD_DO_REPEAT_RELAY <mav_cmd_do_repeat_relay>`
- :ref:`MAV_CMD_DO_DIGICAM_CONFIGURE <mav_cmd_do_digicam_configure>` (Camera enabled only)
- :ref:`MAV_CMD_DO_DIGICAM_CONTROL <mav_cmd_do_digicam_control>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_CAM_TRIGG_DIST <mav_cmd_do_set_cam_trigg_dist>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_ROI <mav_cmd_do_set_roi>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_GIMBAL_MANAGER_PITCHYAW <mav_cmd_do_gimbal_manager_pitchyaw>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_JUMP <mav_cmd_do_jump>`
- :ref:`MAV_CMD_JUMP_TAG <mav_cmd_jump_tag>`
- :ref:`MAV_CMD_DO_JUMP_TAG <mav_cmd_do_jump_tag>`
- :ref:`MAV_CMD_DO_MOUNT_CONTROL <mav_cmd_do_mount_control>`
- :ref:`MAV_CMD_DO_INVERTED_FLIGHT <mav_cmd_do_inverted_flight>`
- :ref:`MAV_CMD_DO_LAND_START <mav_cmd_do_land_start>`
- :ref:`MAV_CMD_DO_FENCE_ENABLE <mav_cmd_do_fence_enable>`
- :ref:`MAV_CMD_DO_AUTOTUNE_ENABLE <mav_cmd_do_autotune_enable>`
- :ref:`MAV_CMD_DO_SET_RESUME_REPEAT_DIST <mav_cmd_do_set_resume_repeat_dist>`
- :ref:`MAV_CMD_STORAGE_FORMAT <mav_cmd_storage_format>`

[/site]

[site wiki="rover" heading="off"]
.. _commands_supported_by_rover:

.. _common-mavlink-mission-command-messages-mav_cmd_commands_supported_by_rover:

Commands supported by Rover
===========================

This list of commands was inferred from the command handler in
`/Rover/commands_logic.cpp <https://github.com/ArduPilot/ardupilot/blob/master/Rover/commands_logic.cpp#L25>`__. 

- :ref:`MAV_CMD_NAV_WAYPOINT <mav_cmd_nav_waypoint>`
- :ref:`MAV_CMD_NAV_RETURN_TO_LAUNCH <mav_cmd_nav_return_to_launch>`
- :ref:`MAV_CMD_NAV_DELAY <mav_cmd_nav_delay>`
- :ref:`MAV_CMD_DO_JUMP <mav_cmd_do_jump>`
- :ref:`MAV_CMD_JUMP_TAG<mav_cmd_jump_tag>`
- :ref:`MAV_CMD_DO_JUMP_TAG <mav_cmd_do_jump_tag>`
- :ref:`MAV_CMD_CONDITION_DELAY <mav_cmd_condition_delay>`
- :ref:`MAV_CMD_CONDITION_DISTANCE <mav_cmd_condition_distance>`
- :ref:`MAV_CMD_DO_AUX_FUNCTION<mav_cmd_do_aux_function>`
- :ref:`MAV_CMD_DO_CHANGE_SPEED <mav_cmd_do_change_speed>`
- :ref:`MAV_CMD_DO_SET_HOME <mav_cmd_do_set_home>`
- :ref:`MAV_CMD_DO_SET_SERVO <mav_cmd_do_set_servo>`
- :ref:`MAV_CMD_DO_SET_RELAY <mav_cmd_do_set_relay>`
- :ref:`MAV_CMD_DO_REPEAT_SERVO <mav_cmd_do_repeat_servo>`
- :ref:`MAV_CMD_DO_REPEAT_RELAY <mav_cmd_do_repeat_relay>`
- :ref:`MAV_CMD_DO_DIGICAM_CONFIGURE <mav_cmd_do_digicam_configure>` (Camera enabled only)
- :ref:`MAV_CMD_DO_DIGICAM_CONTROL <mav_cmd_do_digicam_control>` (Camera enabled only)
- :ref:`MAV_CMD_DO_MOUNT_CONTROL <mav_cmd_do_mount_control>`
- :ref:`MAV_CMD_DO_SET_CAM_TRIGG_DIST <mav_cmd_do_set_cam_trigg_dist>` (Camera enabled only)
- :ref:`MAV_CMD_DO_SET_ROI <mav_cmd_do_set_roi>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_GIMBAL_MANAGER_PITCHYAW <mav_cmd_do_gimbal_manager_pitchyaw>` (Gimbal/mount enabled only)
- :ref:`MAV_CMD_DO_SET_RESUME_REPEAT_DIST <mav_cmd_do_set_resume_repeat_dist>`
- :ref:`MAV_CMD_DO_FENCE_ENABLE <mav_cmd_do_fence_enable>`
- :ref:`MAV_CMD_STORAGE_FORMAT <mav_cmd_storage_format>`

[/site]


.. _common-mavlink-mission-command-messages-mav_cmd_navigation_commands:

Navigation commands
===================

Navigation commands are used to control the movement of the vehicle,
including takeoff, moving to and around waypoints, and landing.

NAV commands have the highest priority. Any DO\_ and CONDITION\_
commands that have not executed when a NAV command is loaded are skipped
(for example, if a waypoint completes and the NAV command for another
waypoint is loaded, and unexecuted DO/CONDITION commands associated with
the first waypoint are dropped.

.. _mav_cmd_nav_waypoint:

MAV_CMD_NAV_WAYPOINT
--------------------

Supported by: All vehicles.

Navigate to the specified position.

[site wiki="copter" heading="off"]

The Copter will fly a straight line to the specified latitude, longitude
, and altitude. It will then wait at the point for a specified delay time
and then proceed to the next waypoint.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Delay</td>
   <td>Hold time at mission waypoint in integer seconds - MAX 65535 seconds.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td>
   </td>
   <td>
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td>
   </td>
   <td>
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td></td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the Copter will hold at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the Copter will hold at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the Copter will hold at the current altitude.</td>
   </tr>
   </tbody>
   </table>


**Mission Planner screenshots**

.. figure:: ../../../images/WayPoint.jpg
   :target: ../_images/WayPoint.jpg

   Copter: Mission Planner Settings for WAYPOINT command
[/site]

[site wiki="plane" heading="off"]

The vehicle will fly to the specified latitude, longitude and altitude.
The waypoint is considered "complete" when Plane is within the specified
radius of the target location, at which point Plane processes the next
command.

The protocol additionally provides for the plane to circle the waypoint
with a specified radius and direction for a specified time (Delay).
These parameters are not supported by Copter.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td>
   </td>
   <td></td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Acc radius</td>
   <td>Acceptance radius in meters (waypoint is complete when the plane is this close to the waypoint location</td>
   </tr>
   <td><strong>param3</strong></td>
   <td>Pass by</td>
   <td>0 to pass through the WP, if > 0 radius in meters to pass by WP.
   Positive value for clockwise orbit, negative value for counter-clockwise
   orbit. Allows trajectory control.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td></td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude</td>
   </tr>
   </tbody>
   </table>
[/site]

[site wiki="rover" heading="off"]


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Delay</td>
   <td>Hold time at mission waypoint in integer seconds - MAX 65535 seconds.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td></td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the Copter will hold at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the Copter will hold at the current longitude.</td>
   </tr>
   <tr>
   <strong>param7</strong>
   <td>Alt</td>
   <td>Target altitude. If zero, the Copter will hold at the current altitude.</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_takeoff:
[site  wiki="copter,plane" heading="off"]

MAV_CMD_NAV_TAKEOFF
-------------------

Supported by: Copter, Plane (not Rover).

Takeoff (either from the ground or by hand-launch). It should be the first
command of nearly all Plane and Copter missions.
[/site]

[site wiki="copter" heading="off"]

The vehicle will climb straight up from its current location to the
specified altitude. If the mission is begun while the copter is already
flying, the vehicle will climb straight up to the specified altitude, if
the vehicle is already above the altitude the command will be ignored
and the mission will move on to the next command immediately.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td>Grade %</td>
   <td>Pitch/climb angle (Plane only).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td>   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td>
   </td>
   <td>Yaw angle (ignored if compass not present).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/TakeOff.jpg
   :target: ../_images/TakeOff.jpg

   Copter: Mission Planner Settings for TAKEOFF command

[/site]

[site wiki="plane" heading="off"]

The plane climbs to the specified altitude (at the specified pitch/climb
angle) before proceeding to the next waypoint.

The plane is *only* attempting to climb at this point and can be pushed
off its heading by wind. The pitch value is the *minimum* climb angle
when using an airspeed sensor, and *maximum* angle when **not**
using an airspeed sensor.

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Pitch Angle</td>
   <td>Pitch/climb angle in degrees</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Yaw angle (ignored if compass not present).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_nav_vtol_takeoff:


MAV_CMD_NAV_VTOL_TAKEOFF
------------------------

Supported by:  Plane  (not Copter or Rover). Specifically QuadPlanes.

Takeoff while in VTOL mode.

The vehicle will climb straight up from it’s current location to the
specified altitude as a delta above its current altitude. 

However, if :ref:`Q_OPTIONS<Q_OPTIONS>` bit 3 is set (use altitude reference frames for VTOL takeoff), then the altitude value (in the specified reference frame) will be used for the target altitude, instead of a delta above the current altitude. If the command is begun while the vehicle is already flying, the vehicle will climb straight up to the specified altitude, if
the vehicle is already above the altitude the command will be ignored
and the mission will move on to the next command immediately.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td>
   </td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>


[/site]

.. _mav_cmd_nav_loiter_unlim:

MAV_CMD_NAV_LOITER_UNLIM
------------------------

Supported by: All vehicles.

Loiter at the specified location for an unlimited amount of time.

[site wiki="copter" heading="off"]

Fly to the specified location and then loiter there indefinitely — where
loiter means "wait in place" (rather than "circle"). If zero is
specified for a latitude/longitude/altitude parameter then the current
location value for the parameter will be used.

The mission will not proceed past this command while in AUTO mode. In
order to break out of this command you need to change the mode (i.e. to
MANUAL). If there are subsequent commands then you can continue the
mission at the next command, if the Copter ``MIS_RESTART`` parameter is
set to resume, by switching back to AUTO mode (otherwise the mission
will restart).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td></td>
   <td>Radius around MISSION, in meters. If positive loiter clockwise, else counter-clockwise</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_LoiterUnlimited.png
   :target: ../_images/MissionList_LoiterUnlimited.png

   Copter: Mission Planner Settings for LOITER_UNLIM command

[/site]

[site wiki="plane" heading="off"]

Fly to the specified location and then loiter there indefinitely — where
loiter means "circle the waypoint". If zero is specified for a
latitude/longitude/altitude parameter then the current location value
for the parameter will be used. You can also specify the radius and
direction for the loiter.

The mission will not proceed past this command while in AUTO mode. In
order to break out of this command you need to change the mode (i.e. to
MANUAL). If there are subsequent commands then you can continue the
mission at the next command, if the Copter ``MIS_RESTART`` parameter is
set to resume, by switching back to AUTO mode (otherwise the mission
will restart).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Dir 1=CW</td>
   <td>Radius around waypoint, in meters. Specify as a positive value to loiter clockwise, as a negative to move counter-clockwise.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_loiter_turns:

[site wiki="copter,plane"]
MAV_CMD_NAV_LOITER_TURNS
------------------------

Supported by: Copter, Plane (not Rover).

[/site]
[site wiki="copter" heading="off"]

Loiter (circle) the specified location for at least the specified number of complete turns,
and then proceed to the next command upon intersection of the course to it with the circle's perimeter. If zero is specified for a latitude/longitude/altitude parameter then the current location value
for the parameter will be used. Fractional turns between
0 and 1 are supported, while turns greater than 1 must be integers.

The radius of the circle is controlled by the
command parameter. A radius of 0 will result in the copter loitering at the location and pirouetting the specified number of turns. Negative radius values result in counter-clockwise turns instead of clockwise turns. Radius values over 255 meters will be rounded down to the nearest 10 meter mark.

This is the command equivalent of the :ref:`Circle flight mode <copter:circle-mode>`.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Turns</td>
   <td>Number of turns (N x 360°)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Radius</td>
   <td>Loiter radius around the waypoint. Units are in meters. Values over 255 will be rounded to units of 10 meters. and values greater than 2550 will be clamped to 2550 m. Negative values indicate counter-clockwise turns. If zero, vehicle will pirouette at location</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_LoiterTurns.png
   :target: ../_images/MissionList_LoiterTurns.png

   Copter: Mission Planner Settings for LOITER_TURNS command

[/site]

[site wiki="plane" heading="off"]

Loiter (circle) the specified location for at least the specified number of complete turns,
and then proceed to the next command upon intersection of the course to it with the circle's perimeter. If zero is specified for a latitude/longitude/altitude parameter then the current location value
for the parameter will be used. Fractional turns between 0 and 1 are supported, while turns greater than 1 must be integers.

The radius of the circle is controlled by the
command parameter. A radius of 0 will result in :ref:`WP_LOITER_RAD<WP_LOITER_RAD>` being used as the radius. Negative radius values result in counter-clockwise turns instead of clockwise turns. Radius values over 255 meters will be rounded down to the nearest 10 meter mark. Instead of exiting at the intersection if the next waypoint's course with the circle's perimeter, a tangential intersection exit point can be selected by setting EXIT =1.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Turns</td>
   <td>Number of turns (N x 360°)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Radius</td>
   <td>Loiter radius around the waypoint. Units are in meters. Values over 255 will be rounded to units of 10 meters. and values greater than 2550 will be clamped to 2550 m. Negative values indicate counter-clockwise turns. A value of zero will use WP_LOITER_RAD </td>
   </tr>
   <td><strong>param4</strong></td>
   <td>Exit</td>
   <td>if 0, exit will occur where path to next waypoint intersects the loiter path after completion of the specified number of turns. if 1, exit will be where next waypoint path is tangential to loiter path </td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

[/site]


.. _mav_cmd_nav_loiter_time:

MAV_CMD_NAV_LOITER_TIME
-----------------------

Supported by: Copter, Plane, Rover.
[site wiki="copter,rover" heading="off"]

Fly/Drive to the specified location and then loiter there for the specified
number of seconds — where loiter means "wait in place" (rather than
"circle"). The timer starts when the waypoint is reached; when it
expires the waypoint is complete. If zero is specified for a
latitude/longitude/altitude parameter then the current location value
for the parameter will be used.

This is the mission equivalent of the :ref:`Loiter flight mode <copter:loiter-mode>` or :ref:`Hold mode <rover:hold-mode>`.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Time s</td>
   <td>Time to loiter at waypoint (seconds - decimal)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td>Dir 1=CW</td>
   <td>Radius around waypoint, in meters. Specify as a positive value to loiter clockwise, as a negative to move counter-clockwise.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_LoiterTime.png
   :target: ../_images/MissionList_LoiterTime.png

   Copter: Mission Planner Settings for LOITER_TIME command

[/site]

[site wiki="plane" heading="off"]

Fly to the specified location and then loiter there for the specified
number of seconds — where loiter means "circle the waypoint". The timer
starts when the waypoint is reached; when it expires the waypoint is
complete. If zero is specified for a latitude/longitude/altitude
parameter then the current location value for the parameter will be
used. You can also specify the radius and direction for the loiter.

The radius of the loiter is set in the ``WP_LOITER_RAD`` parameter.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Time (s)</td>
   <td>Time to loiter at waypoint (seconds - decimal)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Dir 1=CW</td>
   <td>Radius around waypoint, in meters. Specify as a positive value to loiter clockwise, as a negative to move counter-clockwise.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td>
   </td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the vehicle will loiter at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target longitude. If zero, the vehicle will loiter at the current longitude.</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target altitude. If zero, the vehicle will loiter at the current altitude.</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_return_to_launch:

MAV_CMD_NAV_RETURN_TO_LAUNCH
----------------------------

Supported by: All vehicles.

Return to the *home location* or the nearest `Rally
Point <common-rally-points>`__, if closer. The home location is where
the vehicle was last armed (or when it first gets GPS lock after arming
if the vehicle configuration allows this).

[site wiki="copter" heading="off"]


Copter
~~~~~~

Return to the *home location* (or the nearest :ref:`Rally Point <common-rally-points>` if closer) and then land. The home
location is where the vehicle was last armed (or when it first gets GPS
lock after arming if the vehicle configuration allows this).

This is the mission equivalent of the :ref:`RTL flight mode <copter:rtl-mode>`.  The vehicle will
first climb to the
:ref:`RTL_ALT <copter:RTL_ALT>`
parameter's specified altitude (default is 15m) before returning home.

This command takes no parameters and generally should be the last
command in the mission.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_RTL.png
   :target: ../_images/MissionList_RTL.png

   Copter: Mission PlannerSettings for RETURN_TO_LAUNCH command

[/site]

[site wiki="plane" heading="off"]

Return to the *home location* (or the nearest :ref:`Rally Point <common-rally-points>` if closer) and then "Loiter" (circle the
point). The home location is where the vehicle was last armed (or when
it first gets GPS lock after arming if the vehicle configuration allows
this).

If the return is to a rally point, the plane will loiter at the position
and altitude set in the rally point. If the return is to the home
location, then the parameter
:ref:`RTL_ALTITUDE <RTL_ALTITUDE>`
is used for the loiter height (default 100m). The radius of the loiter
is defined in the parameter
:ref:`WP_LOITER_RAD <WP_LOITER_RAD>`.

This command takes no parameters and generally should be the last
command in the mission.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

[site wiki="rover" heading="off"]

Return to the *home location* and HOLD (non-boat) or LOITER (boat).

This command takes no parameters and generally should be the last
command in the mission. Without using this command, end of mission behavior
is set by the :ref:`MIS_DONE_BEHAVE<MIS_DONE_BEHAVE>` parameter.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_land:

[site wiki="copter,plane"]

MAV_CMD_NAV_LAND
----------------

Supported by: Copter, Plane (not Rover).
[/site]
[site wiki="copter"]

The copter will land at its current location or proceed at current altitude to the lat/lon
coordinates provided (if non-zero) and land.  This is the mission equivalent of
the :ref:`LAND flight mode <copter:land-mode>`.

The motors will not stop on their own: you must exit AUTO mode to cut
the engines.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the Copter will land at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Longitude</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_Land.png
   :target: ../_images/MissionList_Land.png

   Copter: Mission Planner Settings for LAND command

[/site]

[site wiki="plane" heading="off"]

The plane will land at its current location or proceed to the (non-zero)
lat/lon coordinates provided beginning with current altitude.  Information on the parameters used to
control the landing is provided in :ref:`LAND flight mode <land-mode>`.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Abort Alt</td>
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude</td>
   </tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude to target for the landing. Unless you are landing at a location different than home, this should be zero</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_nav_vtol_land:

MAV_CMD_NAV_VTOL_LAND
---------------------

Supported by: Plane (not Copter or Rover). Specifically QuadPlanes.

Land the vehicle at the current or a specified location.

If the :ref:`Q_OPTIONS<Q_OPTIONS>` bit 4 is not set (default),the vehicle will land at its current location or proceed at the current altitude to the lat/lon
coordinates provided (if non-zero) and land. The ALT parameter is used to determine final landing phase initiation rather than :ref:`Q_LAND_FINAL_ALT<Q_LAND_FINAL_ALT>`. This is the mission equivalent of the :ref:`QLAND flight mode <qland-mode>`.

If the :ref:`Q_OPTIONS<Q_OPTIONS>` bit 4 is set (Use a fixed wind spiral approach), the it will fly in plane mode to the lat/lon coordinates provided (if non-zero), climbing or descending to the altitude set in the NAV_VTOL_LAND waypoint. When it reaches within :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>` of the landing location, it will perform a LOITER_TO_ALT to finish the climb or descent to that ALT set in the waypoint, then, turning into the wind, transition to VTOL mode and proceed to the landing location and land.

The motors will disarm on their own once landed

.. note:: param1 of the command acts just like :ref:`Q_OPTIONS<Q_OPTIONS>` bit 4 above, if that option bit is not set. This allows the use of different approaches for different VTOL_LAND commands within the same mission.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td></td>
   <td>Option:if set to 1,forces FW spiral approach</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param4</strong></td>
   <td></td>
   <td>Desired yaw angle.</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target latitude. If zero, the QuadPlane will land at the current latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>additional altitude above Q_LAND_FINAL_ALT to switch to final landing phase</td>
   </tr>
   </tbody>
   </table>


[/site]


.. _mav_cmd_nav_continue_and_change_alt:

[site wiki="plane" heading="off"]
MAV_CMD_NAV_CONTINUE_AND_CHANGE_ALT
-----------------------------------

Supported by: Plane (not Copter, Rover).

Continue on the current course and climb/descend to a specified altitude.
Move to the next command when the desired altitude is reached.

.. note::

   The ``param1`` value sets how close the
   vehicle altitude must be to target altitude for command
   completion.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>TBD</td>
   <td>Climb or Descend (0 = Neutral, command completes when within 5m of this
   command's altitude, 1 = Climbing, command completes when at or above
   this command's altitude, 2 = Descending, command completes when at or
   below this command's altitude.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td>param7</td>
   <td>Alt</td>
   <td>Target altitude</td>
   </tr>
   </tbody>
   </table>

[/site]


.. _mav_cmd_nav_spline_waypoint:
[site wiki="copter"]

MAV_CMD_NAV_SPLINE_WAYPOINT
---------------------------

Supported by: Copter (not Plane or Rover).

Fly to the target location using a `Spline path <https://en.wikipedia.org/wiki/Spline_%28mathematics%29>`__, then
wait (hover) for a specified time before proceeding to the next command.

The Spline commands take all the same arguments are regular waypoints
(lat, lon, alt, delay) but when executed the vehicle will fly smooth
paths (both vertically and horizontally) instead of straight lines. 
Spline waypoints can be mixed with regular straight-line waypoints as
shown in the screenshot below.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Delay</td>
   <td>Hold time at target, in decimal seconds.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude/X of goal</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude/Y of goal</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude/Z of goal</td>
   </tr>
   </tbody>
   </table>

.. figure:: ../../../images/MissionList_SplineWaypoint.jpg
   :target: ../_images/MissionList_SplineWaypoint.jpg

   Copter: Mission Planner Settings for SPLINE_WAYPOINT command

The Mission Planner screenshot shows the path the vehicle will take.

-  The 1 second delay at the end of Waypoint #1 causes the vehicle to
   stop so Spline command #2 begins by taking a sharp 90degree turn
-  The direction of travel as the vehicle passes through Spline Waypoint
   #3 is parallel to an imaginary line drawn between waypoints #2 and #4
-  Waypoint #5 is a straight line so the vehicle lines itself up to
   point towards waypoint #5 even before reaching waypoint #4.



.. _mav_cmd_nav_guided_enable:

MAV_CMD_NAV_GUIDED_ENABLE
-------------------------

Supported by: Copter (not Plane or Rover).

Enable ``GUIDED`` mode to hand over control to an external controller/:ref:`common-companion-computers`. ee :ref:`Guided Mode <copter:ac2_guidedmode>` for more information. The :ref:`common-companion-computers`  would then send MAVLink commands to control the vehicle.

See also :ref:`MAV_CMD_DO_GUIDED_LIMITS <mav_cmd_do_guided_limits>`
for information on how to apply time, altitude and distance limits on
the external control.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>on=1/off=0</td>
   <td>A value of > 0.5 enables GUIDED mode. Any value <= 0.5f turns it off.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>
[/site]


.. _mav_cmd_nav_altitude_wait:

[site wiki="plane"]
MAV_CMD_NAV_ALTITUDE_WAIT
-------------------------

Supported by: Plane (not Copter or Rover).

Mission command to wait for an altitude or downward vertical speed.
This is meant for high-altitude balloon launches, allowing the aircraft
to be idle until either an altitude is reached or a negative vertical
speed is reached (indicating early balloon burst). The wiggle time is
how often to wiggle the control surfaces to prevent them from seizing up.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>?</td>
   <td>Altitude (m)</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>?</td>
   <td>Descent speed (m/s)</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>?</td>
   <td>Wiggle Time (s)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>


.. _mav_cmd_nav_loiter_to_alt:

MAV_CMD_NAV_LOITER_TO_ALT
-------------------------

Supported by: Plane (not Copter or Rover).

Loiter while climbing/descending to an altitude.

Begin loitering at the specified Latitude and Longitude. If Lat=Lon=0, then
loiter at the current position. Don't consider the navigation command
complete (don't leave loiter) until the altitude has been reached.
Additionally, if the Heading Required parameter is non-zero the aircraft
will not leave the loiter until heading toward the next waypoint.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Radius</td>
   <td>Radius in meters. If positive loiter clockwise, negative counter-clockwise, 0 means no change to standard loiter.</td>
   </tr>
   <tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_delay:

MAV_CMD_NAV_DELAY
-----------------

Supported by: Copter, Rover, Plane.

[site wiki="copter,rover"]

After reaching this waypoint, delay the execution of the next mission command
until either the time in seconds has elapsed or the time entered(in the future) is reached. Execution of the next mission item
then occurs. For Copters, they will loiter until then, and Rovers hold position.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Time (sec)</td>
   <td>Delay in seconds (integer)</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Time in hours(1-24)</td>
   <td>Delay until this hour</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Time in minutes(0-59)</td>
   <td>Delay until this minute</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>Time in seconds (0-59)</td>
   <td>Delay until this second</td>
   </tr>
   <tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]
[site wiki="plane"]

After reaching this waypoint, if disarmed, delay the execution of the next mission command
until the time in seconds has elapsed. This is used in a mission to allow a vehicle to land, disarm for a period (for a payload change for example), and then re-arm, and takeoff to resume the mission. If not disarmed, this mission item is skipped.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Time (sec)</td>
   <td>Delay in seconds (decimal).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_nav_payload_place:

MAV_CMD_NAV_PAYLOAD_PLACE
-------------------------

Supported by: Copter and Plane.

[site wiki="copter"]

After reaching this waypoint, the vehicle will descend up to the maximum descent value. If the payload has not touched the ground before this limit is reached, the vehicle will climb back up to the waypoint altitude and continue to the next mission item. If it reaches the ground, it will automatically release the gripper if enabled, and optionally wait a period, re-grip, and ascend back to the waypoint altitude and continue the mission. Numerous parameters that control the payload touch-down detection, wait period, etc. are prefaced with ``PLDP_``.
[/site]
[site wiki="plane"]
**QUADPLANE ONLY, fixed wing planes will skip this command**
After reaching this waypoint, the vehicle will have transitioned to VTOL and will descend up to the maximum descent value. If the vehicle has not touched the ground before this limit is reached, the vehicle will climb back up to the waypoint altitude and continue to the next mission item. If it reaches the ground, it will stop its motors and wait for a LUA script command (see `Package Place LUA applet <https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Scripting/applets/plane_package_place.lua>`__ ) to send an abort_landing command to ascend back to the waypoint altitude and continue to the next mission item, be sent a disarm command, or the pilot uses the ``RCx_OPTION`` = 173 to send the abort_landing command, instead of via a LUA script.
This allows the gripper to be commanded to be released, packages replaced, etc.
[/site]
[wiki site="copter,plane"]
**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Maximum Descent</td>
   <td>meters</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude.</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_jump:

MAV_CMD_DO_JUMP
---------------

Supported by: All vehicles.

Jump to the specified command in the mission list. The jump command can
be repeated either a specified number of times before continuing the
mission, or it can be repeated indefinitely.

.. tip::

   Despite the name, this command is really a "NAV\_" command rather
   than a "DO\_" command. Conditional commands like CONDITION_DELAY don't
   affect DO_JUMP (it will always perform the jump as soon as it reaches
   the command).

.. note::

   -  There can be a maximum of 15 jump commands in a mission after which new DO_JUMP commands are ignored.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>WP#</td>
   <td>The command index/sequence number of the command to jump to.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Repeat#</td>
   <td>Number of times that the DO_JUMP command will execute before moving to
   the next sequential command. If the value is zero the next command will
   execute immediately. A value of -1 will cause the command to repeat
   indefinitely.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoJump.png
   :target: ../_images/MissionList_DoJump.png

   Mission Planner Settings for DO_JUMP command

In the example above the vehicle would fly back-and-forth between
waypoints #1 and #2 a total of 3 times before flying on to waypoint #4.

.. _mav_cmd_jump_tag:

MAV_CMD_JUMP_TAG
----------------

Supported by: Copter, Plane, Rover.

This is a location marker in the mission command sequence that can be used as a "jump to" location for the :ref:`MAV_CMD_DO_JUMP_TAG<MAV_CMD_DO_JUMP_TAG>` command. The tag id in its parameter field can be any arbitrary number between 1 and 65535.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Tag#</td>
   <td>The tag number for the DO_JUMP_TAG command.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_jump_tag:

MAV_CMD_DO_JUMP_TAG
-------------------

Supported by: Copter, Plane, Rover.

Jump to the specified :ref:`MAV_CMD_JUMP_TAG<MAV_CMD_JUMP_TAG>` item in the mission list. The jump tag command can be repeated, either a specified number of times before continuing the
mission, or it can be repeated indefinitely.


.. tip::

   Despite the name, this command is really a "NAV\_" command rather
   than a "DO\_" command. Conditional commands like CONDITION_DELAY don't
   affect DO_JUMP (it will always perform the jump as soon as it reaches
   the command).

.. note::

   -  There can be a maximum of 15 jump_tag commands in a mission after which new DO_JUMP_TAG commands are ignored.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>WP#</td>
   <td>The tag number of the JUMP_TAG item to jump to.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Repeat#</td>
   <td>Number of times that the DO_JUMP_TAG command will execute before moving to
   the next sequential command. If the value is zero the next command will
   execute immediately. A value of -1 will cause the command to repeat
   indefinitely.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoJump_Tag.png
   :target: ../_images/MissionList_DoJump_Tag.png

   Mission Planner Settings for DO_JUMP_TAG command

   In the example above, the vehicle would fly back-and-forth between waypoints #3 and #4 a total of 3 times before completing the mission. This is because the DO_JUMP_TAG redirects the vehicle to JUMP_TAG #565 twice.

Conditional commands
====================

Conditional commands control the execution of \_DO\_ commands. For
example, a conditional command can prevent DO commands from executing based
on a time delay, until the vehicle is at a certain altitude, or at a
specified distance from the next target position.

A conditional command may not be complete before reaching the next
waypoint. In this case, any unexecuted \_DO\_ commands associated with
the last waypoint will be skipped.

.. _mav_cmd_condition_delay:

MAV_CMD_CONDITION_DELAY
-----------------------

Supported by: All vehicles.

After reaching a waypoint, delay the execution of the next conditional
"_DO_" command for the specified number of seconds (e.g.
:ref:`MAV_CMD_DO_SET_ROI <mav_cmd_do_set_roi>`).

.. note::

   This command does not stop the vehicle. If the vehicle reaches the
   next waypoint before the delay timer completes, the delayed "_DO_"
   commands will never trigger.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Time (sec)</td>
   <td>Delay in seconds (decimal).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_ConditionDelay.png
   :target: ../_images/MissionList_ConditionDelay.png

   Mission Planner Settings for CONDITION_DELAY command

In the example above, Command #4 (``DO_SET_ROI``) is delayed so that it
starts 5 seconds after the vehicle has passed Waypoint #2.


.. _mav_cmd_condition_distance:

MAV_CMD_CONDITION_DISTANCE
--------------------------

Supported by: All vehicles.

Delays the start of the next "``_DO_``\ " command until the vehicle is
within the specified number of meters of the next waypoint.

.. note::

   This command does not stop the vehicle: it only affects DO
   commands.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Dist (m)</td>
   <td>Distance from the next waypoint before DO commands are executed (meters).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_ConditionDistance.png
   :target: ../_images/MissionList_ConditionDistance.png

   Mission PlannerSettings for CONDITION_DISTANCE command

In the example above, Command #4 (``DO_SET_ROI``) is delayed so that it
only starts once the vehicle is within 50m of waypoint #5.


.. _mav_cmd_condition_yaw:

[site wiki="copter" heading="off"]

MAV_CMD_CONDITION_YAW
---------------------

Supported by: Copter (not Plane or Rover).

Point (yaw) the nose of the vehicle towards a specified heading.

The parameters allow you to specify whether the target direction is
absolute or relative to the current yaw direction. If the direction is
relative you can also (separately) specify whether the value is added or
subtracted from the current heading (note that the vehicle will always
turn in the direction that most quickly gets it to the new target heading
regardless of the ``param3`` value).


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Deg</td>
   <td>
   If <code>param4=0</code> (absolute): Target heading in degrees [0-360] (0 is North).

   If <code>param4=1</code> (relative): The change in heading (in degrees).
   </td>
   </tr>
   <td><strong>param2</strong></td>
   <td>Speed deg/s</td>
   <td>Speed during yaw change:[deg per second].</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Dir 1=CW</td>
   <td>Used to denote the direction of rotation to achieve the target angle (-1=CCW, 1=CW, 0=the vehicle will always turn in the direction that most quickly gets it to the new target heading, but only if <code>param4=0</code> (absolute), otherwise 0 = CW).
   </td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>0=Abs 1=Rel</td>
   <td>Specify if <code>param1</code> ("Deg" field) is an absolute direction (0) or a relative to the current yaw direction (1).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_ConditionYaw.png
   :target: ../_images/MissionList_ConditionYaw.png

   Copter: Mission Planner Settings for CONDITION_YAW command

[/site]
[site wiki="copter" heading="off"]

Special Commands
================

This section is for commands that may be relevant to missions, but
which are not mission commands (part of the mission).

.. _mav_cmd_mission_start:

MAV_CMD_MISSION_START
---------------------

Supported by: Copter

This command can be used to start a mission when the Copter is on the
ground in AUTO mode. If the vehicle is already in the air then the
mission will start as soon as you switch into AUTO mode (so this command
is not needed/ignored). This allows a GCS/companion computer
to start a mission in AUTO without raising the throttle.

.. note::

   Previously a mission would only start after the pilot engaged the
   throttle. This command makes it possible to start missions without
   directly controlling the throttle (though that approach is still
   available).

This is not a "mission command" (it can't be used as a type of mission
waypoint). It is run from the **Action** menu (see the screenshot
below).

**Command parameters**

The parameters are all ignored.

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>

   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td></td>
   <td>The first mission item to run.</td>
   </tr>

   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td></td>
   <td>The last mission item to run (after this item is run, the mission ends).</td>
   </tr>

   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td></td>
   </tr>

   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td></td>
   </tr>

   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td></td>
   </tr>

   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td></td>
   </tr>

   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td></td>
   </tr>
   </tbody>
   </table>

**Mission Planner screenshots**

.. figure:: ../../../images/MissionPlanner_MissionStartCommand.jpg
   :target: ../_images/MissionPlanner_MissionStartCommand.jpg

   Mission Planner: MISSION_START command

.. _mav_cmd_component_arm_disarm:

MAV_CMD_COMPONENT_ARM_DISARM
----------------------------

Supported by: Copter

Disarm the motors.

The command supports disarming on the ground and in flight.

.. note::

   The motors will disarm automatically after landing.

This is not a "mission command" (it can't be used as a type of mission
waypoint).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td></td>
   <td>1 to arm, 0 to disarm. This only works when the vehicle is on the ground.
   </td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td></td>
   <td>A value of 21196 will disarm the vehicle in flight.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td></td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td></td>
   </tr>
   </tbody>
   </table>
[/site]

DO commands
===========

The "DO" or "Now" commands are executed once to perform some action. All
the DO commands associated with a waypoint are executed immediately.



.. _mav_cmd_do_change_speed:

MAV_CMD_DO_CHANGE_SPEED
-----------------------

Supported by: Copter, Plane, Rover.

[site wiki="copter" heading="off"]

Sets the desired maximum speed in meters/second (only). Both the
speed-type and throttle settings are ignored.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Type</td>
   <td>Speed type (0,1=Ground Speed,  2=Climb Speed, 3=Descent Speed).</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>speed in m/s</td>
   <td>Target speed (m/s).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoChangeSpeed.png
   :target: ../_images/MissionList_DoChangeSpeed.png

   Copter: Mission Planner Settings for DO_CHANGE_SPEED command

[/site]

[site wiki="plane" heading="off"]

Change the target horizontal speed (airspeed or groundspeed) and/or the
vehicle's throttle. If the airspeed option is selected, this changes the :ref:`AIRSPEED_CRUISE<AIRSPEED_CRUISE>` parameter during the flight until reboot or mode is changed to CRUISE or FBWB. If the groundspeed option is used, then :ref:`MIN_GROUNDSPEED<MIN_GROUNDSPEED>` parameter is changed to this value until rebooted or changed by this command again. If the throttle field is non-zero and equal to or below 100, then the :ref:`TRIM_THROTTLE<TRIM_THROTTLE>` parameter is changed  until reboot or changed by this command again.

.. note:: Speed changes only have an effect if an airspeed sensor is present, healthy, and in use. :ref:`TRIM_THROTTLE<TRIM_THROTTLE>` changes impacts only flight with airspeed sensor not in use.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Type </td>
   <td>Speed type (0=Airspeed, 1=Ground Speed).</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Speed (m/s)</td>
   <td>Target speed (m/s). If airspeed, a value below or above min/max airspeed limits results in no change. a value of -2 uses :ref:`AIRSPEED_CRUISE<AIRSPEED_CRUISE>`</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Throttle(%)</td>
   <td>Throttle as a percentage (0-100%). A value of 0 or negative indicates no change.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

[site wiki="rover" heading="off"]

Change the target horizontal speed and/or the vehicle's throttle.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Speed (m/s)</td>
   <td>Target speed (m/s). </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_set_home:

MAV_CMD_DO_SET_HOME
-------------------

Supported by: All vehicles.

Sets the home location either as the current location or at the location
specified in the command. For SITL work, altitude input here needs to be with reference to absolute altitude, taking into account SRTM elevation.

.. note::

   -  For Plane and Rover, if a good GPS fix cannot be obtained the
      location specified in the command is used.
   -  For Copter, the command will also try to use the current position if
      all the location parameters are set to 0. The location information in
      the command is only used if it is close to the EKF origin.

[site wiki="copter" heading="off"]

[/site]

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Current</td>
   <td>Set home location:

   1=Set home as current location.

   0=Use location specified in message parameters.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Target home latitude (if <code>param1=0</code>)</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Lon</td>
   <td>Target home longitude (if <code>param1=0</code>)</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Target home altitude (if <code>param1=0</code>)</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoSetHome.png
   :target: ../_images/MissionList_DoSetHome.png

   Mission Planner Settings for DO_SET_HOME command

.. _mav_cmd_do_set_relay:

MAV_CMD_DO_SET_RELAY
--------------------

Supported by: All vehicles.

Set a `Relay <common-relay>`__ pin's voltage high (on) or low (off).


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Relay No</td>
   <td>Relay number.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>off(0)/on(1)</td>
   <td>Set relay state:

   1: Set relay high/on (3.3V on Pixhawk, 5V on APM).

   0: Set relay low/off (0v)
   any other value toggles the relay
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionPlanner_DO_SET_RELAY.png
   :target: ../_images/MissionPlanner_DO_SET_RELAY.png

   MissionPlanner Settings for DO_SET_RELAY command

.. _mav_cmd_do_repeat_relay:

MAV_CMD_DO_REPEAT_RELAY
-----------------------

Supported by: All vehicles.

Toggle the :ref:`Relay <common-relay>` pin's voltage/state a specified
number of times with a given period. Toggling the Relay will turn an off
relay on and vice versa

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Relay No</td>
   <td>Relay number.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Repeat #</td>
   <td>Cycle count - the number of times the relay should be toggled</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Delay(s)</td>
   <td>Cycle time (seconds, decimal) - time between each toggle.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoRepeatRelay.png
   :target: ../_images/MissionList_DoRepeatRelay.png

   Mission Planner Settings for DO_RELAY_REPEAT command

In the example above, assuming the relay was off to begin with, it would
be set high and then after 3 seconds, it would be toggled low again.

.. _mav_cmd_do_set_servo:

MAV_CMD_DO_SET_SERVO
--------------------

Supported by: All vehicles.

Set a given :ref:`servo pin <common-servo>` output to a specific PWM value.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Ser No</td>
   <td>Servo number - target servo output pin/channel number.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>PWM</td>
   <td>PWM value to output, in microseconds (typically 1000 to 2000).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoSetServo.png
   :target: ../_images/MissionList_DoSetServo.png

   Mission Planner Settingsfor DO_SET_SERVO command

In the example above, the servo attached to output channel 8 would be
moved to PWM 1700 (servos generally accept PWM values between 1000 and
2000).

.. note:: as of firmware versions 4.0 and later, this command can be used on any output configured by its ``SERVOx_FUNCTION`` command as 0,1, or 51-66  (disabled or RC pass-throughs)

.. _mav_cmd_do_repeat_servo:

MAV_CMD_DO_REPEAT_SERVO
-----------------------

Supported by: All vehicles.

Cycle a :ref:`servo <common-servo>` PWM output pin between its mid-position
value and a specified PWM value, for a given number of cycles and with a
set period.

The mid-position value is specified in the ``RCn_TRIM`` parameter for
the channel (``RC8_TRIM`` in the screenshot below). The default value is
1500..

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Ser No</td>
   <td>Servo number - target servo output pin/channel.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>PWM</td>
   <td>PWM value to output, in microseconds (typically 1000 to 2000).</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Repeat #</td>
   <td>Cycle count - number of times to move the servo to the specified PWM value</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>Delay (s)</td>
   <td>Cycle time (seconds) - the delay in seconds between each servo movement.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoRepeatServo.png
   :target: ../_images/MissionList_DoRepeatServo.png

   Mission Planner Settingsfor DO_REPEAT_SERVO command

In the example above, the servo attached to output channel 8 would be
moved to PWM 1700, then after 4 second, back to mid, after another 4
seconds it would be moved to 1700 again, then finally after 4 more
seconds it would be moved back to mid.

.. _mav_cmd_do_land_start:

[site wiki="plane" heading="off"]

MAV_CMD_DO_LAND_START
---------------------

Supported by: Plane (not Copter, Rover).

Mission command to prepare for a landing.

This is used as a marker in a mission to tell the autopilot where a
sequence of mission items that represents a landing starts. It may also
be sent via a ``COMMAND_LONG`` to trigger a landing, in which case the
nearest (geographically) landing sequence in the mission will be used.

If ``RTL_AUTOLAND`` is set to 2, the plane will jump to the nearest 
``DO_LAND_START`` in the mission table when RTL is initialized. 

.. note::

   General information on landing a plane is provided in the topic
   :ref:`Automatic Landing <automatic-landing>`.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param1</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude used to help find the closest landing sequence, or zero if not needed.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude used to help find the closest landing sequence, or zero if not needed.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_vtol_transition:

MAV_CMD_DO_VTOL_TRANSITION
--------------------------

Supported by: Plane (not Copter or Rover).Specifically QuadPlanes.

Mission command to change to/from VTOL and fixed wing mode of flight. The mode is changed based on the first parameter: 3 = change to VTOL flight, 4 = change to fixed wing flight.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Mode</td>
   <td>3=VTOL,4=Fixed-Wing</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td>
   </td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_set_roi:

MAV_CMD_DO_SET_ROI
------------------

Supported by: Copter, Plane, Rover.
[site wiki="copter" heading="off"]

Points the :ref:`camera gimbal <common-cameras-and-gimbals>` at the "region
of interest", and also rotates the nose of the vehicle if the
mount type does not support a yaw feature.

After setting the ROI, the camera/vehicle will continue to follow it
until the end of the mission, unless it is changed or cleared by setting
another ROI. Clearing the ROI is achieved by setting a later
DO_SET_ROI command with all zero for ``param5``-``param7`` (Lat, Lon
and Alt).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td></td>
   <td>Region of interest mode. (see MAV_ROI enum) // 0 = no roi, 1 = next
   waypoint, 2 = waypoint number, 3 = fixed location, 4 = given target (not supported)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td></td>
   <td>MISSION index/ target ID. (see MAV_ROI enum)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td></td>
   <td>ROI index (allows a vehicle to manage multiple ROI's)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude (x) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude (y) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude of the fixed ROI</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots/video**

.. figure:: ../../../images/MissionList_DoSetRoi.jpg
   :target: ../_images/MissionList_DoSetRoi.jpg

   Copter: Mission Planner Settings for DO_SET_ROI command

In the example above the nose and camera would be pointed at the red
marker.

..  youtube:: W8NCFHrEjfU
    :width: 100%

[/site]

[site wiki="plane" heading="off"]

Points the :ref:`camera gimbal <common-cameras-and-gimbals>` at the "region
of interest".

After setting the ROI, the camera will continue to follow it until the
end of the mission, unless it is changed or cleared by setting another
ROI. Clearing the ROI is achieved by setting a later DO_SET_ROI
command with all zeros for ``param5``-``param7`` (Lat, Lon and Alt).


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td></td>
   <td>Region of interest mode. (see MAV_ROI enum) // 0 = no roi, 1 = next
   waypoint, 2 = waypoint number, 3 = fixed location, 4 = given target (not supported)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td>
   </td>
   <td>MISSION index/ target ID. (see MAV_ROI enum)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td></td>
   <td>ROI index (allows a vehicle to manage multiple ROI's)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude (x) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude (y) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude of the fixed ROI</td>
   </tr>
   </tbody>
   </table>

[/site]

[site wiki="rover" heading="off"]

Points the :ref:`camera gimbal <common-cameras-and-gimbals>` at the "region
of interest".

After setting the ROI, the camera will continue to follow it until the
end of the mission, unless it is changed or cleared by setting another
ROI. Clearing the ROI is achieved by setting a later DO_SET_ROI
command with all zeros for ``param5``-``param7`` (Lat, Lon, and Alt).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param1</strong></td>
   <td></td>
   <td>Region of interest mode. (see MAV_ROI enum) // 0 = no roi, 1 = next
   waypoint, 2 = waypoint number, 3 = fixed location, 4 = given target (not supported)
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td></td>
   <td>MISSION index/ target ID. (see MAV_ROI enum)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td></td>
   <td>ROI index (allows a vehicle to manage multiple ROI's)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Lat</td>
   <td>Latitude (x) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param6</strong></td>
   <td>Long</td>
   <td>Longitude (y) of the fixed ROI</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Alt</td>
   <td>Altitude of the fixed ROI</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_digicam_configure:

MAV_CMD_DO_DIGICAM_CONFIGURE
----------------------------

Supported by: All vehicles.

Configure an on-board camera controller system.

The parameters are forwarded to an on-board camera controller system
(like the `3DR Camera Control Board <common-camera-control-board>`__),
if one is present.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Mode</td>
   <td>Set camera mode:
   1: ProgramAuto

   2: Aperture Priority

   3: Shutter Priority

   4: Manual

   5: IntelligentAuto

   6: SuperiorAuto
   </td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Shutter Speed</td>
   <td>Shutter speed (seconds divisor). So if the speed is 1/60 seconds, the
   value entered would be 60. Slowest shutter trigger supported is 1 second.
   </td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>Aperture</td>
   <td>Aperture: F stop number</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>ISO</td>
   <td>ISO number e.g. 80, 100, 200, etc.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param5</strong></td>
   <td>ExposureMode</td>
   <td>Exposure type enumerator</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>CommandID</td>
   <td>Command Identity</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td>Engine Cut-Off</td>
   <td>Main engine cut-off time before camera trigger in seconds/10 (0 means no cut-off).</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_digicam_control:

MAV_CMD_DO_DIGICAM_CONTROL
--------------------------

Supported by: All vehicles.

Trigger the :ref:`camera shutter <common-camera-shutter-with-servo>` once. This command takes no additional arguments.

**Command parameters**

In general, if a command field is set to 0 it is ignored.

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>On/Off</td>
   <td>Session control (on/off or show/hide lens):

   0: Turn off the camera / hide the lens

   1: Turn on the camera /Show the lens
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param2</strong></td>
   <td>Zoom Position</td>
   <td>Zoom's absolute position. 2x, 3x, 10x, etc.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param3</strong></td>
   <td>Zoom Step</td>
   <td>Zooming step value to offset zoom from the current position</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>Focus Lock</td>
   <td>Focus Locking, Unlocking or Re-locking:
   0: Ignore

   1: Unlock

   2: Lock
   </td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td>Shutter Cmd</td>
   <td>Shooting Command. Any non-zero value triggers the camera.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td><strong>param6</strong></td>
   <td>CommandID</td>
   <td>Command Identity</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoDigicamControl.png
   :target: ../_images/MissionList_DoDigicamControl.png

   Mission PlannerSettings for DO_DIGICAM_CONTROL command.

.. _mav_cmd_do_mount_control:

MAV_CMD_DO_MOUNT_CONTROL
------------------------

Supported by: All vehicles.

Mission command to control a camera or antenna mount.

This command allows you to specify a roll, pitch and yaw angle which
will be sent to the :ref:`camera gimbal <common-cameras-and-gimbals>`. This
can be used to point the camera in specific directions at various times
in the mission.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td></td>
   <td>Pitch, in degrees.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td></td>
   <td>Roll, in degrees.</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td></td>
   <td>Yaw, in degrees.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>reserved</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>reserved</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>reserved</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td></td>
   <td>`MAV_MOUNT_MODE <https://mavlink.io/en/messages/common.html#MAV_MOUNT_MODE>`__ enum value.</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoMountControl.png
   :target: ../_images/MissionList_DoMountControl.png

   Mission Planner Settings for DO_MOUNT_CONTROL command

.. _mav_cmd_do_gimbal_manager_pitchyaw:

MAV_CMD_DO_GIMBAL_MANAGER_PITCHYAW
----------------------------------

Supported by: All vehicles.

Mission command to move the gimbal to the desired pitch and yaw angles (in degrees).

This command allows you to specify a pitch and yaw angle which
will be sent to the :ref:`camera gimbal <common-cameras-and-gimbals>`. This
can be used to point the camera in specific directions at various times
in the mission. Positive pitch angles are up, Negative are down.  Positive yaw angles are clockwise, negative are counter clockwise.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td></td>
   <td>Pitch, in degrees.</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td></td>
   <td>Yaw, in degrees.</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td></td>
   <td>PitchRate, in deg/s.</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td></td>
   <td>YawRate, in deg/s</td>
   </tr>
   <tr>
   <td><strong>param5</strong></td>
   <td></td>
   <td>Flags: 0=boddyframe, 16=earthframe</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>reserved</td>
   </tr>
   <tr>
   <td><strong>param7</strong></td>
   <td></td>
   <td> Gimbal instance ID</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/mission-list-do-gimbal-manager-pitchyaw.png
   :target: ../_images/mission-list-do-gimbal-manager-pitchyaw.png

   Mission PlannerSettings for DO_GIMBAL_MANAGE_PITCHYAW command

.. _mav_cmd_do_set_cam_trigg_dist:

MAV_CMD_DO_SET_CAM_TRIGG_DIST
-----------------------------

Supported by: All vehicles.

Trigger the :ref:`camera shutter <common-camera-shutter-with-servo>` at
regular distance intervals. This command is useful in :ref:`camera survey missions <common-camera-control-and-auto-missions-in-mission-planner>`. 
To trigger the camera once, immediately after passing the DO command, set param3 to 1.  Trigger immediately Parameter is available from ArduPilot 4.1 onwards.

.. note::

   Providing a distance of zero will stop the camera shutter from being triggered.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Dist (m)</td>
   <td>Camera trigger distance interval (meters). Zero to turn off distance triggering.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <td><strong>param3</strong></td>
   <td>?</td>
   <td>Trigger once instantly. One is on, zero is off.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

**Mission planner screenshots**

.. figure:: ../../../images/MissionList_DoSetCamTriggDist.png
   :target: ../_images/MissionList_DoSetCamTriggDist.png

   Mission PlannerSettings for DO_SET_CAM_TRIGG_DIST command

The above configuration will cause the camera shutter to trigger
after every 5m that the vehicle travels.

.. _mav_cmd_do_fence_enable:


MAV_CMD_DO_FENCE_ENABLE
-----------------------

Supported by: All vehicles.

Mission commands to enable the Plane :ref:`GeoFence <geofencing>`, Copter/Rover :ref:`common-ac2_simple_geofence` and/or :ref:`common-polygon_fence`.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td></td>
   <td>Set GeoFence enable state (0=disable, 1=enable, 2= disable only floor (Plane only)).</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>bitmask</td>
   <td>The target fence is specified by the bitmask value of FENCE_TYPE. 0 is ALL configured fences.</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_aux_function:

MAV_CMD_DO_AUX_FUNCTION
-----------------------

Supported by: All vehicles.

Mission command to control an :ref:`Auxiliary Function<common-auxiliary-functions>` in the same manner as an RC channel switch.


**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Aux Function</td>
   <td>Auxiliary Function code,same as RCx_OPTIONS </td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>Switch Position</td>
   <td> 0:Low, 1:Mid, 2:High</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_parachute:
[site wiki="copter" heading="off"]

MAV_CMD_DO_PARACHUTE
--------------------

Supported by: Copter (not Plane or Rover).

Mission command to trigger a parachute (if enabled).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Enable/Release</td>
   <td>Parachute action (0=disable, 1=enable, 2=release).</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]


.. _mav_cmd_do_inverted_flight:

[site wiki="plane" heading="off"]

MAV_CMD_DO_INVERTED_FLIGHT
--------------------------

Supported by: Plane (not Copter or Rover).

Change between normal and :ref:`inverted flight <inverted-flight>`.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>0=normal, 1=inverted</td>
   <td>Set flight type:

   0: normal
   1: inverted
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_gripper:

[site wiki="copter" heading="off"]

MAV_CMD_DO_GRIPPER
------------------

Supported by: Copter (not Plane or Rover).

Mission command to operate EPM gripper.

.. note::

   The :ref:`instructions for integrating Copter with gripper <common-electro-permanent-magnet-gripper>` are out
   of date and use DO_SET_SERVO to activate the gripper (April 2015).

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Gripper No</td>
   <td>Gripper number (from 1 to maximum number of grippers on the vehicle). </td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>drop(0)/grab(1)</td>
   <td>Gripper action:
   0:Release
   1:Grab
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_guided_limits:

MAV_CMD_DO_GUIDED_LIMITS
------------------------

Supported by: Copter (not Plane or Rover).

This command sets the time, altitude, and distance limits for external
control (GUIDED mode). When these limits are exceeded, control will
return from GUIDED mode to the mission. Setting any of these parameters
to zero will remove the associated limit.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>timeout S</td>
   <td>Maximum time (in seconds) that the external controller is allowed to
   control vehicle. Use 0 to remove time limits (unlimited time allowed).
   </td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>min alt</td>
   <td>Minimum allowed absolute altitude (in meters, AMSL), below which the
   command will be aborted and the mission will continue. Use 0 to indicate
   that there is no minimum altitude limit.
   </td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>max alt</td>
   <td>Maximum allowed absolute altitude (in meters, AMSL), above which the
   command will be aborted and the mission will continue. Use 0 to indicate
   that there is no maximum altitude limit.
   </td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>max dist</td>
   <td>Horizontal move limit (in meters, AMSL). If the vehicle moves more than this
   distance from its location at the moment the command was executed, the
   the command will be aborted and the mission will continue. Use 0 to indicate
   that there is no horizontal limit.
   </td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_autotune_enable:

[site wiki="plane" heading="off"]

MAV_CMD_DO_AUTOTUNE_ENABLE
--------------------------

Supported by: Plane (not Copter or Rover).

Enable/disable :ref:`AUTOTUNE <autotune-mode>` mode.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>na</td>
   <td>Enable/disable autotune (1: enable, 0:disable)</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_do_engine_control:

MAV_CMD_DO_ENGINE_CONTROL
-------------------------

Supported by: Plane (not Copter or Rover).

Stop or start the internal combustion engine (ICE)

This command can be used to start or stop the ICE before a NAV_VTOL_LAND or after a NAV_VTOL_TAKEOFF command for a QuadPlane to avoid potential prop strikes in the wind. It should be placed before either of those commands.Also can be used to allow a single engine start while disarmed if otherwise prohibited by :ref:`ICE_OPTIONS<ICE_OPTIONS>` bit 3 being set,

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>?</td>
   <td>Start/Stop ICE (1: start, 0:stop)</td>
   </tr>
   <td><strong>param2</strong></td>
   <td></td>
   <td>Cold Start (1: enables choke, currently not implemented)</td>
   </tr>
   <td><strong>param3</strong></td>
   <td></td>
   <td>Altitude in meters. Altitude at which action is taken.</td>
   </tr>
   <td><strong>param4</strong></td>
   <td></td>
   <td>Flags: 1 = allow a single start while disarmed even if :ref:`ICE_OPTIONS<ICE_OPTIONS>` bit 3 is set</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[/site]

.. _mav_cmd_do_set_resume_repeat_dist:

MAV_CMD_DO_SET_RESUME_REPEAT_DIST
---------------------------------

Supported by: All vehicles.

Set the distance that the mission will be rewound when resuming after an interrupt (switching modes).  A full explanation of this feature can be found on the :ref:`Mission Rewind on Resume Page <common-mission-rewind>`.  After setting a rewind distance in a mission, setting the distance to zero will switch off the rewind feature from that point on the mission.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>?</td>
   <td>Rewind distance in meters</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param2</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

.. _mav_cmd_storage_format:

MAV_CMD_STORAGE_FORMAT
----------------------

Supported by: All vehicles.

Format SD Card. Useful for vehicles where SD card is inaccessible. Param1 and Param2 must be set to 1.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>?</td>
   <td>Must be 1</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>?</td>
   <td>Must be 1</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param3</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param4</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>

[site wiki="copter" heading="off"]
.. _mav_cmd_do_winch:

MAV_CMD_DO_WINCH
----------------

Supported by: Copter.

Control Winch operation.

**Command parameters**

.. raw:: html

   <table border="1" class="docutils">
   <tbody>
   <tr>
   <th>Command Field</th>
   <th>Mission Planner Field</th>
   <th>Description</th>
   </tr>
   <tr>
   <td><strong>param1</strong></td>
   <td>Winch no</td>
   <td>Not currently used</td>
   </tr>
   <tr>
   <td><strong>param2</strong></td>
   <td>action</td>
   <td>0 to relax the winch, 1 for Length control, 2 for Rate control</td>
   </tr>
   <tr>
   <td><strong>param3</strong></td>
   <td>length</td>
   <td>should be filled in with the meters of the line to release.  Positive numbers release the line, negative retract the line.  Note "action" should be "1"</td>
   </tr>
   <tr>
   <td><strong>param4</strong></td>
   <td>rate</td>
   <td>should be filled in with the speed (in m/s) to release the line.  Positive numbers release the line, negative retract the line.  Note "action" should be "2".</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param5</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param6</td>
   <td></td>
   <td>Empty</td>
   </tr>
   <tr style="color: #c0c0c0">
   <td>param7</td>
   <td></td>
   <td>Empty</td>
   </tr>
   </tbody>
   </table>
[/site]

[copywiki destination="plane,copter,rover,sub,planner,dev"]
