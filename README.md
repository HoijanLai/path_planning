## Path Planning

---

### Introduction
This is a path planning project for solving a lane changing problems for self-driving car. 

### Result

link: https://youtu.be/DnvvD-Ixwh4

Behaviors include:
* Staying on its lane
* Switching lane when needed
* Smoothly following planning waypoints
* No collision[*]

_*not fully tested_

### Method
* Logic
  There're three main target of the car in this problem:

  * To staying its lane when there's no other cars near ahead
  * To keeping an relatively high speed
  * To avoid any collision

  Target 1 is pretty easy to realize. To solve the second problem, the ego car should be able to change lane when the car upfront is running slow. In this case, the car should surpass the car upfront to keep desired speed, which degrades to a simple lane changing problem. 

  In the simulator, the car is guided by a series of planning waypoints given by the program, so lane changing corresponds to switching the latest waypoints to adjacent lanes when it is safe to do so.

* Finite states machine

  The car decides whether it is safe to change lane by looking at the sensor fusion data, which is basically about where are the other cars locate and how they move.

  A lane changing is allowed when the conditions below are fulfilled:

  * The car is not current switching lane
  * There's at least one sufficient gap between two cars in a row on the desired lane.
  * The desired gap is on a comfortable location[*]. 

  _*no support for an initiative lane change yet_

  * The car is correctly aligned with a gap

  Here are the code for state transitions:

  ```c++
  double safe_gap = 20.0;
  double gap_l = s_ul - s_ll; // s_ul denotes the s coordinate of the car upfront on the left lane (upper left) 
  double gap_r = s_ur - s_lr;

  bool changing = fabs(4*lane+2 - car_d) > 1.0; // whether the ego car is changing lane
  cout << "changing: " << changing << endl;

  bool l_ok = lane > 0 && gap_l >= safe_gap && s_ul >= 10 && s_ll <= -5; 

  bool r_ok = lane < 2 && gap_r >= safe_gap && s_ur >= 10 && s_lr <= -5; 

  if (changing) car_state = KL;
  else {
    if      (l_ok &&  r_ok) 
      if (gap_l > gap_r) car_state = LCL;
      else               car_state = LCR; 
    
    else if (l_ok && !r_ok) car_state = LCL;
    else if (r_ok && !l_ok) car_state = LCR; 
    else                    car_state = KL; 
  }
  ```

   



* Spline

  A spline is fitted to some knots we set including the newly visited ones and  the ones for realizing the latest state.

  The plan starts with a series of waypoints that guides the car to keep its current lane. The number of waypoints are fixed. The car realizes the first couple of them in every timestep and in the next timestep, some new waypoints are added to the end of the plan to keep the length of the plan. 

  The spline functionality is responsible for smoothing the plan and preventing jerking. It works like magic and saves me some cost functions stuff.

  Reference: http://kluge.in-chemnitz.de/opensource/spline/

### Improvement
* Initiative Lane Change

  For now, the finite states machine includes only three states: keep lane, lane change left and lane change right. It is sufficient to drive safely, but is a bit passive. More states should be included for initiative lane change, for example, the car should be able to decelerate and align with a comfortable gap instead of waiting a gap to come.

* Emergency Handling

  RPCs in the simulator seems to be really responsible on road. In reality, some drivers will act pretty unpredictable like taking immediate brakes or taking lane change without looking at the rearview. 

* Reinforcement Learning 

  The core idea of this project is to write some rules for state transitions. But rules are endless because the environment is way more complicated. I found this project is pretty similar to the Deep Traffic playground by MIT. Hopefully we can introduce the neural network in the future.

  Reference: http://selfdrivingcars.mit.edu/deeptrafficjs/
