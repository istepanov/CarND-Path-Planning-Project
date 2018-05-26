# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

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

## Reflection

### Keeping the lane

To keep the car on the middle of current lane, I do next steps:

1. Calculate 3 distant points 30, 60 and 90 meters ahead of the car - so basically add these distances to current `car_s` position.
Because lane width is 4 metes, `d` value calculates as `4 * lane + 2`, where `lane` is current lane number (`0` - left most lane, `2` - right most lane).
Notice I add 2 meters to stay in the middle. To convert `(s, d)` into `(x, y)`, I used `getXY()` function. _(lines 326 - 334)_

2. In addition to these 3 points, I use points calculated on previous step that the car hasn't reached yet (if not enough points,
infer them using car's position and orientation). _(lines 298 - 323)_

3. All these points must be converted to car reference system. _(lines 342-349)_

4. Next, calculate spline using `spline.h` library. _(lines 352-353)_

5. Use the spline to calculate intermediate points. The trick is to keep these points in such distance so it take 20ms for the car to get from one point to another.
_(see variable `N` on line 373)_ Also those points need to be converted back to absolute coordinates. _(lines 383-384)_

6. Finally, send generated points back to the simulator. _(lines 394-400)_

### Avoiding collisions

The car uses sensor fusion data to calculate if a collision is going to happen.

First, it estimates if a target car is located on the same lane, then estimates target car position at the time our car finishes
going through existing path points. If target car position is too close, the car slows down by 0.224 mph per iteration
(which is provide the acceleration about 5 m/s^2). If there's no car in front of us, try to gain speed until speed limit is reached
(again, adding 0.224 mph per iteration). _(lines 236-288)_

### Lane changing

In addition to slowing down to avoid collision, the car can change current lane to pass a slower car in front of it. Logic is simple:

1. If current lane is not a left most lane, check if there's a car on the lane to the left (the safety window is 20m behind and 30m ahead).
Again, a predicted target car position is used, not a current position.

2. If it's safe to do a lane change maneuver, do so by simply changing current lane value - spline calculation will do the rest! (see _Keeping the lane_ section above).

3. If left lane is busy, perform the same actions for a lane right to the car (if it's not right most lane, of cause). _(lines 236-288)_
