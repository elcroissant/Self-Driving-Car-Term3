# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Rubric Points

1. The code compiles correctly.
```
The In order to compile the project use ./install-mac.sh in main directory. It will create build directory where you can find path_planning binary.
-- The C compiler identification is AppleClang 9.0.0.9000039
-- The CXX compiler identification is AppleClang 9.0.0.9000039
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
[ 50%] Building CXX object CMakeFiles/path_planning.dir/src/main.cpp.o
[100%] Linking CXX executable path_planning
[100%] Built target path_planning
```
2. The car is able to drive at least 4.32 miles without incident..
<img src="data/Screen Shot 2018-04-03 at 1.01.30 AM.png" width="480" alt="Combined Image" />

3. The car drives according to the speed limit.
The code is using traffic cost class (lines main.cpp:171-269) as the cost function for lane changes.
The constructor of the class gets 50 mph initially as speed limit. 
```
TrafficCost cost(lane, 50);
```
The speed limit then (inside traffic cost class) may change if we have slower car ahead of us
and we can't change lane (see line main.cpp:221)

After processing all records of sensor fusion table we retrieve that speed limit from 
the traffic cost instance just before we are deciding whether to accelerate or slow down
```
main.cpp:410
double max_current_vel = cost.getMaxVelocity();
```
4. Max Acceleration and Jerk are not Exceeded.

The potential acceleration is controled in the code as follows:
```
main.cpp:411-421
if (max_current_vel - ref_vel > 0) {
   if (max_current_vel - ref_vel < .5) {
      ref_vel = max_current_vel;
   }
   else {
      ref_vel += .224; // ~5m/s
   }
}
else if (max_current_vel - ref_vel < 0) {
   ref_vel -= .224;
}
```
5. Car does not have collisions.
In order to avoid collisions program implements several measures, such as:
```
main.cpp:219-221
if (car_lane == car_detected_lane && car_detected_vel < max_velocity){
   // decrease max allowed velocity if another car is ahead of us
   max_velocity = car_detected_vel;
```

```
main.cpp:241-247
if((((car_detected_pos < car_pos) && ((car_pos - car_detected_pos) < gap_rear)) ||
    ((car_detected_pos > car_pos) && ((car_detected_pos - car_pos) < gap_front_next))) &&
     (car_lane != car_detected_lane)) {
      // don't change line when car is on neighbor lane
      // and so so far behind us or next to us
      costs[car_detected_lane] += high_penalty;
}
```
```
main.cpp:259-264
if (cost_ready == true)
{
   cout <<  costs[0] << " " << costs[1] << " " << costs[2] << endl;
  // if we are changing lane then we don't have to slow the car down
   if (car_lane != getLane()) {
      max_velocity = speed_limit;
```
etc

6. The car stays in its lane, except for the time between changing lanes.
The car initially starts from lane 1
```
main.cpp:181-187
// init cost with low penalty
      costs[0] = low_penalty;
      costs[1] = low_penalty;
      costs[2] = low_penalty;
// zero lane occupied by the car so that the car
// won't change one lane to another with no reason
      costs[current_lane] = 0;
}
```
Then during drive updateCost main.cpp:211-268 is used to verify if it is good to change lane or not.

Additionaly if the change lane is in progress the car can't change to another line in the middle

```
main.cpp:249-257
if (change_in_progress)
{
   // if we decided to change lane to anodher one,
   // when still changing from previous lane
   costs[0] = high_penalty;
   costs[1] = high_penalty;
   costs[2] = high_penalty;
   costs[car_lane] = 0;
}
```

7. The car is able to change lanes
The car is able to change lane when the cost function says it's better to do so.
```
main.cpp:200-203
int getLane() {
   // get index for the lowest cost
   return distance(costs.begin(), min_element(costs.begin(), costs.end()));
}
```

8. Reflection on how to generate paths.

The path planning algorithm begins when we get previous list of points:
```
main.cpp:252-254
// previous list of points - the last path that the car was following before 
// this particuar run through of calculating more points
int prev_size = previous_path_x.size();
```
Phase no 1 is about prediction and it is when the car is processing sensor fusion data
to understand the envirnoment and decide whether it is safier to slow down and stay on 
the current lane or change it. The code is using TrafficCost class to produce 
costs function and based on that the car is making appropriate maneuver. 

If the detected car is ahead us within 60m and on different lanes we are adding typical cost
for single car (here is just 1). Also we are adding penalty proportionally to the difference of 
velocities of our car and car ahead. To increase chance to chane to less occupied lane we also 
add penalty which is reverse proportionally to normalized distance between our car and car ahead.

If the detected car is ahead us 30m and on our current lane and our max allowed velocity is 
bigger than car detected velocity then we are deacreasing our velocity so that we won't crash
with the car ahead. Also we are adding cost for that car equal to 1 and increase it by adding 
penalty proportionally to the difference of velocities of our car and car ahead.
```
main.cpp:215-239
if((car_detected_pos > car_pos) && ((car_detected_pos - car_pos) < 2 * gap_front)) {

         if((car_detected_pos > car_pos) && ((car_detected_pos - car_pos) < gap_front)) {

            if (car_lane == car_detected_lane && car_detected_vel < max_velocity){
               // decrease max allowed velocity if another car is ahead of us
               max_velocity = car_detected_vel;
             // add typical cost for single car
               costs[car_detected_lane] += cost;
               // add penalty proportionally to the difference of velocities of
               // our car and car ahead
               costs[car_detected_lane] += (car_vel - car_detected_vel) / 100;
            }
         }
         if (car_lane != car_detected_lane){
            // add typical cost for single car
           costs[car_detected_lane] += cost;
            // add penalty proportionally to the difference of velocities of
            // our car and car ahead 
           costs[car_detected_lane] += (car_vel - car_detected_vel) / 100;
            // add penalty in reverse proportion to normalized distance between our car and car ahead
            costs[car_detected_lane] += 1 - (car_detected_pos - car_pos) / (2 * gap_front);
            assert((car_detected_pos - car_pos) / (2 * gap_front) < 1);
         }
   }
```

If the detected car is on neighbor lane and within 11m behind or right next to us (10m ahead) then 
we are adding panalty for that lane.
```
main.cpp:241-247
if((((car_detected_pos < car_pos) && ((car_pos - car_detected_pos) < gap_rear)) ||
    ((car_detected_pos > car_pos) && ((car_detected_pos - car_pos) < gap_front_next))) &&
     (car_lane != car_detected_lane)) {
      // don't change line when car is on neighbor lane
      // and so so far behind us or next to us
      costs[car_detected_lane] += high_penalty;
}
```

If we decided to change the lane but we are just in the middle of previous change 
we are blocking that maneuver. 

```
main.cpp:249-257
if (change_in_progress)
{
   // if we decided to change lane to anogher one,
   // when still changing from previous lane
   costs[0] = high_penalty;
   costs[1] = high_penalty;
   costs[2] = high_penalty;
   costs[car_lane] = 0;
}
```

And finally when we have all sensor funsion process and we know we will change the lane 
then we don't need to slow down

```
main.cpp:259-267
if (cost_ready == true)
{
   cout <<  costs[0] << " " << costs[1] << " " << costs[2] << endl;
  // if we are changing lane then we don't have to slow the car down
   if (car_lane != getLane()) {
      max_velocity = speed_limit;
      //cout << "update max_velocity " << max_velocity << endl;
   }
}
```

Phase no 2 is about behavior. After predition is finished we know whether we are going to change lane or not:

```
main.cpp:401
lane = cost.getLane();
```

And then we are deciding whether we should accelerate or slow down or keep max_current velocity
If we have reached max_current velocity then we press break and slow down by ~5m/s, otherwise we 
increase speed by ~5m/s unless we are very close to our max_current velocity and in that case we are 
trying to keep velocity of the car ahead or a bit below speed limit.
```
main.cpp:410-421
double max_current_vel = cost.getMaxVelocity();
if (max_current_vel - ref_vel > 0) {
   if (max_current_vel - ref_vel < .5) {
      ref_vel = max_current_vel;
   }
   else {
      ref_vel += .224; // ~5m/s
   }
}
else if (max_current_vel - ref_vel < 0) {
   ref_vel -= .224;
}
```

Phase no 3 is about trajectory. main.cpp:434-543. It uses calculated speed and lane from previous phase, coordinates of the car and previous points to calculate trajectory. 

We first creating 5 anchor points and later we will interpolate these points with the spline and fill it in with more points that control spline. Our first tow anchor poinst are poinst where the car is or previous path's end points. Three left anchor points are three evenly 30m spaced poinst ahead of starting reference main.cpp:473-475. 

Then, we transform those points to local car's coordinates by shifting and rotating and then create a spline. Then we calculate how to break up spline points so that we travel at desired reference velocity main.cpp:516-519 and fill up the rest of our planner path and finally we rotate all points back to global coordinates


## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!


## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

