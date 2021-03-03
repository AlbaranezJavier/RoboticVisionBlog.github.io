# Robotic memories
In this web I will solve and comment on the practices developed in the Robotic Vision subject. This subject belongs to the Master's degree in Computer Vision of the URJC.
I am Javier Albaráñez Martínez, a student of this university master's program.

## Practices
1. **Follow line**, this practice consists of modeling artificial intelligence capable of controlling an autonomous car. To do this, we have at our disposal an environment modeled virtually on the Unibotics website.

## 1. Follow line

### Day 1: Familiarization with the environment
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

The results obtained were 10 minutes, but by making some adjustments it was reduced to 4 minutes.

Well, the results are not good at all, now it's time to implement the steps marked by our teacher in the Robotic Vision subject.

### Day 2: Basic control
