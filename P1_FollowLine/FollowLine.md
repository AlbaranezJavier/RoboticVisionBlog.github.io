# Follow line

## Index
+ [Familiarization with the environment](#familiarization-with-the-environment)
+ [Basic control](#basic-control)
+ [Proportional controller P](#proportional-controller-p)
+ [Controller D to controller PD](#controller-d-to-controller-pd)
+ [Controller I to controller PID](#controller-i-to-controller-pid)
+ [Where is the line?](#where-is-the-line)
+ [Endurance test](#endurance-test)
+ [Reverse](#reverse)
+ [Drive safe](#drive-safe)
+ [Conclusions](#conclusions)
## Familiarization with the environment
The first thing to do was to set up the practice environment, for this I used the following [website](https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/). It describes the necessary steps to set up the website, although some sections are outdated. Our teacher took care of detailing the missing steps.

1. Clone the Robotics Academy repository on your local machine.
```
git clone https://github.com/JdeRobot/RoboticsAcademy
```
2. Download [Docker](https://docs.docker.com/get-docker/).
3. Pull the current distribution of Robotics Academy Docker Image.
```
docker pull jderobot/robotics-academy
```
4. On the local machine navigate to the follow_line exercise which is: RoboticsAcademy/exercises/follow_line/web-template/assets/websocket_address.js, change the variable websocket_address to the IP address through which the container is connected. Usually for Linux machine it is 127.0.0.1 and for Windows is 192.168.99.100.

5. Start a new docker container of the image and keep it running in the background.
```
docker run -it -p 8080:8080 -p 7681:7681 -p 2303:2303 -p 1905:1905 -p 8765:8765 jderobot/robotics-academy python3.8 manager.py
```
6. Sign up and sign in at [Unibotics](https://unibotics.org/).
7. Go to the exercise [follow line](https://unibotics.org/academy/login?next=/academy/exercise/follow_line/).

In order to understand how the environment works and to look for a first solution, I have implemented a solution simply with what I could think of. 

Selecting 5 equispaced columns in the image, I counted the number of pixels represented in a mask. This mask is only collecting the reds of the image in HSV format. 

Going back to the 5 columns, they represent how much of that line is in them, so if I have values on the right I know I should rotate to the right, but instead I calculate the difference between the values left on the right and calculate the difference with the central value. This is the error I make in deciding whether to turn right or turn left.

In the case of the accelerator, I use the central value of those 5 columns to decide whether to accelerate more or less.

The results obtained were 10 minutes, but by making some adjustments it was reduced to 4 minutes, but I have been able to extract a lot of information on how to attack this problem:
- The car does not suffer from sway when accelerating, braking or turning.
- The red line is continuous and there is no other object present in the environment that could hinder its detection.
- There are no level changes, so the detection can be done with a prefixed line.
- In addition, we have seen some examples where the error can be seen as it is extracted with a vertical line as a reference.

Well, the results are not good at all, now it's time to implement the steps marked by our teacher in the Robotic Vision subject.

## Basic control
The objective is to implement a right turn or left turn switch, depending on the error obtained with a reference vertical column. In addition, some deadlash is applied to prevent it from continuously rectifying near the equilibrium point.

This implementation has a strong zig zag component, this is because the basic controller stops driving the turn in one direction when it has passed the optimum point.

To solve this type of problem, I have resorted to resetting the rotation when it is between the optimal solution. But in addition, there was the bug that if the artificial intelligence starts correcting only with the horizon, it starts to advance the gyro earlier and therefore, a goal of this practice is not satisfied. The main objectives are:

- To stay above the red line.
- To complete the circuit in the shortest possible time.

So, what I have done is to add another line closer to the point of view of the car and I have controlled the importance of each one by setting a weight, therefore they will not affect in the same way the turn of the car.

Then I added memory so it will be able to remember which way it was turning. The next step is to implement a proportional controller, this will allow me to reduce the zig zag effect.

<p align="center">
  <img src="Videos/P1_1.gif" alt="video basic control" width="65%" />
</p>

## Proportional controller P
This controller is based on adding a Kp constant in order to weight the effect of the error.

Implementing this type of controller, there are several differences with the previous one. In this case, how much the vehicle will rotate is determined proportionally by the error measurement obtained.

To calculate this error, I have changed the implementation and now calculate the centroid of the image for a range. This allows me to obtain a more robust implementation of the problem, since I am not limited to a single horizontal line of the camera.

Anyway, I have adjusted the available parameters so that the lap time is as short as possible, so for other circumstances this may not be the best configuration.

The lap time has been reduced to between 36 - 37s. I want to mention the help of a classmate, because in the original formula the value obtained in the previous instant was added, but this made the response worse. When this parameter was eliminated, the times were greatly reduced. 

<p align="center">
  <img src="Videos/ControllerP.gif" alt="video basic control" width="65%" />
</p>

## Controller D to controller PD

In this step, I am going to add a new constant to the controller (Kd). This component is the derivative of the error, which translates into the difference between the previous error and the current error. This component is added with the proportional controller. The expected consequence of this component is that if the error continues to increase from one instant to the next, the correction applied by the proportional component will be intensified. The opposite is true for the opposite case. In other words, the response will be intensified if the correction is not sufficient and will be damped if it approaches the optimum.

The results have not been long in coming and this has greatly increased the speed of the vehicle. The oscillations produced by the proportional controller have been greatly reduced.

In addition, in this step the results obtained by the proportional and derivative controller have been used to control the acceleration of the vehicle. The logic behind this design decision is that one expects to accelerate when the error is small, but to brake drastically when there are sudden variations in the track (such as when arriving at a curve from a straight line).

The results obtained have improved, the lap time is between 23-24 s, but as you can see in the video below, there are quite a few oscillations.

<p align="center">
  <img src="Videos/ControllerPD.gif" alt="video basic control" width="65%" />
</p>

## Controller I to controller PID

This is the latest version to be explored. In it appears the constant Ki which multiplies the integral of the error with respect to time. This means that the error will accumulate and its effects are as follows:
- When a curve step is being performed, this component is forcing a correction in the direction of the curve, compensating for the error that P cannot rectify.
- On the other hand, if this component is well parameterized, it allows damping the oscillations, but if it is too large it will intensify them.

For this controller, it has been decided to change the way of controlling the accelerator pedal. Now, it starts from a specific speed and is reduced according to the difference between the angle of rotation of the steering wheel at this instant and the previous one. This is intended to make the car accelerate when it is in a stable position and brake in the opposite case.

The central reference of the camera has also been modified, moving it a little to the right, since the camera is not centered with the car.

What has been achieved with this controller is to improve the lap time by keeping the car on the line in a reasonable way (23 - 24s).

<p align="center">
  <img src="Videos/ControllerPID.gif" alt="video basic control" width="65%" />
</p>

## Where is the line?

Another requirement of the practice is to manage the following situation: What happens if the car starts in a position where there is no line? 
To solve this, before entering the execution loop, you are going to create an instruction that applies a turn in a given direction until a line is found.

The purpose of this section is to increase the robustness of the system in certain situations. The main difficulty encountered was that the car detected a line and started to drive at maximum speed, to avoid this, the maximum speed is reached after chasing the red line for a certain period of time. Another not quite correct behavior, is that when the car detects the red line again it acts like a high pressure spring, so it does not seem very natural.

<p align="center">
  <img src="Videos/Noline.gif" alt="video basic control" width="65%" />
</p>

## Endurance test
In order to verify that the proposed system is robust enough for this circuit, a test of 9 consecutive laps has been carried out. 

Some faults detected in the simulation are: the chronometer stops working correctly and the fps of the GUI decreases during the simulation, which makes it difficult to test the system.
<p align="center">
  <img src="Videos/9laps.gif" alt="video basic control" width="65%" />
</p>

## Reverse

I also wanted to check how the car behaves in counterclockwise direction, the lap time is similar to the one obtained in clockwise direction. (24 - 25s)

<p align="center">
  <img src="Videos/reverse.gif" alt="video basic control" width="65%" />
</p>

## Drive safe
In the latter experiment, the requirement to go above the line is sought to be met. At the same time, I try to go as fast as possible. The lap time has been between 28-29s.

In the video below, you can see the final version of the experiment with a couple of laps around the circuit.

To achieve this goal, I have reduced the maximum speed and all other constants.

<p align="center">
  <img src="Videos/drivesafe.gif" alt="video basic control" width="65%" />
</p>

## Conclusions

The conclusions obtained during the experimentation carried out with the practice are:

- P, PD and PID controllers, have very good behaviors, with little coding and can be tuned experimentally.
- The P controller indicates how much to correct the error, if the Kp constant is too high the oscillations increase and if it is too little, the error is not corrected enough.
- The D controller corrects these oscillations, but if it is too high it can increase them.
- The controller I corrects the error that PD cannot, but if it is too high this correction ends up acting too long.
- In the end you have to find the balance between speed and how much you are on the line, in my case I think I have opted a little more for the speed. This is because if the speed is increased, the frames available to make decisions is reduced and forces to have a faster response, which ends up translating into a system that seems to be governed by a high pressure spring, with quite abrupt reactions.
- The experiments carried out to find good parameters can take a long time, because it is played with 3 constants, speed range, how to estimate the error, etc.. 
- The simulation environment makes the task much easier, although using the same parameters on several computers the results are different.

As a final conclusion, it is completely clear to me that you can create behaviors that require very little compute and are very fast. Also that finding a solution is simple, but trying to adjust the parameters to obtain the best solution is very complicated. On the other hand, it requires a fairly controlled environment to work properly.

A very interesting practice! I have learned a lot!

| [Go up](#follow-line) | [Main page](../index.md) |
