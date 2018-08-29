===============================
Async Motion Profile Controller
===============================

.. contents:: :local:

okapi::AsyncMotionProfileController
===================================

An ``AsyncController`` which generates and follows 2D motion profiles.

This controller is internally backed by `Pathfinder <https://github.com/JacisNonsense/Pathfinder>`_. There are a few open issues users should be aware about:

- Pathfinder cannot generate negative velocities, so backward movements and very tight turns do not work
    - Moving backwards: `<https://github.com/JacisNonsense/Pathfinder/issues/39>`_
    - Tight turns: `<https://github.com/JacisNonsense/Pathfinder/issues/38>`_
- Very long movements (typically movements much longer than a VEX field) can potentially never reach maximum speed
    - `<https://github.com/JacisNonsense/Pathfinder/issues/43>`_

Constructor(s)
--------------

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        AsyncMotionProfileController(const TimeUtil &itimeUtil,
                                     double imaxVel, double imaxAccel, double imaxJerk,
                                     std::shared_ptr<ChassisModel> imodel, Length iwidth)

=============== ===================================================================
 Parameters
=============== ===================================================================
 itimeUtil       See ``TimeUtil`` docs.
 imaxVel         The maximum possible velocity.
 imaxAccel       The maximum possible acceleration.
 imaxJerk        The maxiumm possible jerk.
 imodel          The ``ChassisModel`` to control.
 iwidth          The chassis' wheelbase width.
=============== ===================================================================

Methods
-------

generatePath
~~~~~~~~~~~~

Generates a path which intersects the given waypoints and saves it internally with a key of pathId.
Call executePath() with the same pathId to run it.

If the waypoints form a path which is impossible to achieve, an instance of ``std::runtime_error``
is thrown (and an error is logged) which describes the waypoints.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void generatePath(std::initializer_list<Point> iwaypoints, std::string ipathId)

============ ===============================================================
 Parameters
============ ===============================================================
 iwaypoints   The waypoints to hit on the path.
 ipathId      A unique identifier to save the path with.
============ ===============================================================

----

setTarget
~~~~~~~~~

Executes a path with the given ID. If there is no path matching the ID, the method will return.
Any targets set while a path is being followed will be ignored.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void setTarget(std::string ipathId) override

============ ===============================================================
 Parameters
============ ===============================================================
 ipathId      A unique identifier for the path, previously passed to ``generatePath()``.
============ ===============================================================

----

getTarget
~~~~~~~~~

Gets the last set target, or the default target if none was set.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        std::string getTarget() override

**Returns:** The last target.

----

waitUntilSettled
~~~~~~~~~~~~~~~~

Blocks the current task until the controller has settled. This controller is settled when it has
finished following a path. If no path is being followed, it is settled.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void waitUntilSettled() override

----

getError
~~~~~~~~

Returns the last error of the controller. This implementation always returns zero since the robot
is assumed to perfectly follow the path. Subclasses can override this to be more accurate using
odometry information.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        Point getError() const override

**Returns:** The last error of the controller.

----

isSettled
~~~~~~~~~

Returns whether the controller has settled at the target. Setting is when the error or derivative
of error has been small enough for a long enough period.

If the controller is disabled, this method must return true.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        bool isSettled() override

**Returns:** Whether the controller is settled.

----

reset
~~~~~

Resets the controller so it can start from 0 again properly. Keeps configuration from before.

This implementation does nothing.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void reset() override

----

flipDisable
~~~~~~~~~~~

Changes whether the controller is off or on. Turning the controller on after it was off will NOT
cause the controller to move to its last set target, unless it was reset in that time.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void flipDisable() override

----

flipDisable
~~~~~~~~~~~

Sets whether the controller is off or on. Turning the controller on after it was off will NOT
cause the controller to move to its last set target, unless it was reset in that time.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void flipDisable(bool iisDisabled) override

============= ===============================================================
 Parameters
============= ===============================================================
 iisDisabled   Whether the controller should be disabled.
============= ===============================================================

----

isDisabled
~~~~~~~~~~

Returns whether the controller is currently disabled.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        bool isDisabled() override

**Returns:** Whether the controller is currently disabled.

----

startThread
~~~~~~~~~~~

Starts the internal thread. This should not be called by normal users. This method is called by the
``AsyncControllerFactory`` when making a new instance of this class.

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        void startThread()

----

okapi::Point
============

.. tabs ::
   .. tab :: Prototype
      .. highlight:: cpp
      ::

        struct Point {
            QLength x;    // X coordinate relative to the start of the movement
            QLength y;    // Y coordinate relative to the start of the movement
            QAngle theta; // Exit angle relative to the start of the movement
        };