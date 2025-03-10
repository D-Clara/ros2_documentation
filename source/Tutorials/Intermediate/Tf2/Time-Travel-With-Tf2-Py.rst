.. redirect-from::

    Tutorials/Tf2/Time-Travel-With-Tf2-Py

.. _TimeTravelWithTf2Py:

Traveling in time (Python)
==========================

**Goal:** Learn about advanced time travel features of tf2.

**Tutorial level:** Intermediate

**Time:** 10 minutes

.. contents:: Contents
   :depth: 2
   :local:

Background
----------

In the previous tutorial, we discussed the :doc:`basics of tf2 and time <./Learning-About-Tf2-And-Time-Py>`.
This tutorial will take us one step further and expose a powerful tf2 trick: the time travel.
In short, one of the key features of tf2 library is that it is able to transform data in time as well as in space.

This tf2 time travel feature can be useful for various tasks, like monitoring the pose of the robot for a long period of time or building a follower robot that will follow the "steps" of the leader.
We will use that time travel feature to look up transforms back in time and program ``turtle2`` to follow 5 seconds behind ``carrot1``.

Time travel
-----------

First, let's go back to where we ended in the previous tutorial :doc:`Using time <./Learning-About-Tf2-And-Time-Py>`.
Go to your ``learning_tf2_py`` package.

Now, instead of making the second turtle go to where the carrot is now, we will make the second turtle go to where the first carrot was 5 seconds ago.
Edit the ``lookup_transform()`` call in ``turtle_tf2_listener.py`` file to

.. code-block:: python

    when = self.get_clock().now() - rclpy.time.Duration(seconds=5.0)
    trans = self._tf_buffer.lookup_transform(
        to_frame_rel,
        from_frame_rel,
        when,
        timeout=Duration(seconds=0.05))

Now if you run this, during the first 5 seconds, the second turtle would not know where to go because we do not yet have a 5-second history of poses of the carrot.
But what happens after these 5 seconds? Let's just give it a try:

.. code-block:: console

    ros2 launch learning_tf2_py turtle_tf2_fixed_frame_demo.launch.py

.. image:: images/turtlesim_delay1.png

You should now notice that your turtle is driving around uncontrollably like in this screenshot. Let's try to understand reason behind that behavior.

#. In our code we asked tf2 the following question: "What was the pose of ``carrot1`` 5 seconds ago, relative to ``turtle2`` 5 seconds ago?". This means we are controlling the second turtle based on where it was 5 seconds ago as well as where the first carrot was 5 seconds ago.

#. However, what we really want to ask is: "What was the pose of ``carrot1`` 5 seconds ago, relative to the current position of the ``turtle2``?".

Advanced API for lookup_transform()
-----------------------------------

To ask the tf2 that particular question, we will use an advanced API that gives us the power to say explicitly when to acquire the specified transformations.
This is done by calling the ``lookup_transform_full()`` method with additional parameters.
Your code now would look like this:

.. code-block:: python

    when = self.get_clock().now() - rclpy.time.Duration(seconds=5.0)
    trans = self._tf_buffer.lookup_transform_full(
            target_frame=to_frame_rel,
            target_time=rclpy.time.Time(),
            source_frame=from_frame_rel,
            source_time=when,
            fixed_frame='world',
            timeout=Duration(seconds=0.05))

The advanced API for ``lookup_transform_full()`` takes six arguments:

#. Target frame

#. The time to transform to

#. Source frame

#. The time at which source frame will be evaluated

#. Frame that does not change over time, in this case the ``world`` frame

#. Time to wait for the target frame to become available

To sum up, tf2 does the following in the background.
In the past, it computes the transform from the ``carrot1`` to the ``world``.
In the ``world`` frame, tf2 time travels from the past to now.
And at the current time, tf2 computes the transform from the ``world`` to the ``turtle2``.

Checking the results
--------------------

Let's run the simulation again, this time with the advanced time-travel API:

.. code-block:: console

    ros2 launch learning_tf2_py turtle_tf2_fixed_frame_demo.launch.py

.. image:: images/turtlesim_delay2.png

And yes, the second turtle is directed to where the first carrot was 5 seconds ago!

Summary
-------

In this tutorial, you have seen one of the advanced features of tf2.
You learned that tf2 can transform data in time and learned how to do that with turtlesim example.
tf2 allowed you to go back in time and make frame transformations between old and current poses of turtles by using the advanced ``lookup_transform_full()`` API.
