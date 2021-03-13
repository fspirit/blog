---
toc: true
layout: post
categories: [motion_planning]
title: Motion Planning for Self-Driving Cars
---

## Motion Planning for Self-Driving Cars

In this series of posts I would like to answer the question: How would we build motion planning component for a self-driving car from scratch using current state of the art algorithms.

*Disclaimer*: I'm not a professional researcher  working on this topic, I'm a software developer in the automotive industry, so I will look at different approaches mostly from the perspective of how hard are they to implement and will that implementation lead too satisfying functional & non-functional (runtime) performance. With that in mind, some lack of math/theory can be expected. 

First of all let us define a place and function of Motion Planning component/subsystem in the the whole self-driving stack. In the end it must answer a question - where the car goes next 
and produce a trajectory from which actuator values, acceleration and steering, can be derived and passed to the Control block.

The input is basically all the data we can gather about environment and our ego car: road data (road geometries, signs), ego odometry data (position, heading, steering angle and rate of change of those values), max allowed and desired speeds, data about other traffic participants (current poses ans predicted trajectories) and obstacles.

As output the component must produce a trajectory, which must be:
- Safe, a risk of collision with other vehicles or obstacles should be as low as possible
- Feasible, a car should be able to drive resulting trajectory. For instance, maximum turning angle should be considered
- Comfortable, a driver and/or passengers should feel comfortable while a car drives them. For example, lateral acceleration (which depends on the velocity of the vehicle and curvature of the path it follows) must not exceed a certain limit

Usually Motion Planning task is decomposed into 3 stages, which also usually come in order, producing input for the next stage, although for different use cases and/or decision making approaches some of them can be skipped or merged.

![]({{ site.baseurl }}/images/MotionPlanning.png)

### Route Planning 

At this stage we look at road graph and, knowing the final destination, construct a path through road segments to reach it. Here the graph search algorithms are employed, Dijkstra, A\*, D\* and the like.
The result is a sequence of road segments ego vehicle has to follow.

### Maneuver Planning

Here we look at what kind of manuevers the car can perform to reach the next road segment from the route generated at the previous stage.
Examples for highway scenario can be: keep current lane, change to left, change to right.
Then, depending on approach, we either select one immediately and proceed to generating trajectory or
we generate trajectory for each of possible maneuvers and select one of them, based on some set of criteria.
We consider wide range of factors, like road limits (current speed limit, traffic lights, lane merging/splitting).
Most popular approches here include:
- State machines
- Maneuver selection based on cost
- Reinforcement learning (including Deep RL)

### Trajectory Planning

The remark should be made about path and trajectory distinction:
while path contains just geometrical information and is an ordered sequence of poses (x, y, heading, curvature) of the vehicle, a trajectory 
considers time dimension and thus will also entail velocities and accelerations.
One way to get final trajectory is to generate collision-free, feasible path first and then decide on velocity profile to maximize comfort. Another approach would be to try to state and solve a problem, which takes velocities and accelerations into account right away.
Methods here include:
- **Graph-based**. First generate a graph of possible states a vehicle can be in next X seconds, for instance, divide all the space around into geometric shapes like squares or rectangles, assigning cost to each, or using several possible feasible movements recusively build a tree. Then run on top of that one of those graph search algorithms mentioned already: Dijkstra, A\*, D\* or their variations.
- **RTT family**. This algorithms randomly samples next pose for the vehicle and builds a tree, until it reaches area around target pose. 
- **Numerical optimization**. Here the problem of generating trajectory is stated as an optimal control problem, that is as a cost functional to minimize (which can take different forms), set of variables that can be controlled (acceleration/steering), quality and inequality constraints on those and derived variables. Then we run solvers to get the target variables values, which minimize cost functions and satisfy all the constraints.
- **Curve Interpolation**. Take a set of fine-grained waypoints (another question is who will generate them) and fit a curve with desired properties using appropriate curve types: splines, spirals, clothoids, Bezier curves and so on.

In the next post I'll go into details of Trajectory Planning step, digging deeper into one or several of the approaches and discussing how they achieve safety/feasibility and comfort properties and how easy to implement/fast are they.


