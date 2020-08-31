---
layout: post
title: "adaptive cruise control with PID control"
---

## goal

Adaptive cruise control (ACC) is a vehicle control system that modifies the speed of the ego vehicle in response to conditions on the road. The ACC operates in two modes: speed control and distance control. The control goal varies depending on which mode is set. The goal of the system is to maintain the cruise speed with speed control, whilst also ensuring a safe distance between the ego vehicle and the lead vehicle is maintained. Another constraint on the system is minimum and maximum acceleration, to ensure for any hard brake (for example) the driver does not experience high Gs. 


## system modeling

The entire system consists of two vehicles: an ego vehicle and a lead vehicle. Both vehicles can be regarded as acting independently with a control force (external acceleration) applied through some heuristic.

This could be done with more rigor with regards to the dynamic model, but maybe another time. Nonetheless, assuming that the vehicle acts purely in a one dimensional plane, either traveling forwards or backwards, by way of an applied force. This vehicle is assumed to be a point mass with some frontal area and the ability to have rolling friction with the ground. As such it can be expressed as

$$ m \frac{dv(t)}{dt} = F\sb{control} - \frac{1}{2} \rho A v(t)^2 C\sb{D} - C\sb{rr} m g $$

Where the forces are the sum of the engine (applied through the controller) force, air resistance, and rolling resistance. Not much more detail is required as the system is incredibly simple and the controller output influences the acceleration which in turn updates the velocity and position. These are both updated according to

$$ v = v + a * dt $$

and

$$ p = p + v * dt $$

where dt is some timestep.

## method

The lead vehicle acceleration was determined to oscillate sinusoidally in an attempt to repeatedly trigger the adaptive cruise control of the ego vehicle. The control force for the lead vehicle in implementation was bypassed, and the acceleration was set manually. It can be assumed that the effects of mass, air resistance, and rolling resistance are bundled into the sinusoidal acceleration value. This was manually set using 

```cpp
Vehicle::UpdateStateManual(acceleration);
```

The sinusoid was generated with the formula


```cpp
acceleration = AMPLITUDE*sin(M_PI*CYCLES*i/SIMDURATION);
```

Where AMPLITUDE was 2, M\_PI was the pi constant, CYCLES was the number of cycles over the entire duration which was 5, i was the clock value (iterator), and SIMDURATION was the duration of the simulation.

The ego vehicle acceleration was determined through a control value returned by the PID controller through

```cpp
Vehicle::UpdateState(acc.ApplyControl(lead.GetPosition(), ego.GetPosition(), ego.GetVelocity()));
```

As expected the `.GetPosition()` and `.GetVelocity()` methods return the position of the vehicle from the origin and the velocity of the vehicles, respectively. The `acc.ApplyControl()` method supplies the adaptive cruise control with the ability to find the distance between the lead and ego vehicles to ascertain if the set distance has been violated, and the ego relative velocity to ascertain whether the set velocity has been violated.

If the set distance is not violated, the control value is computed with the PID controller with 

```cpp
control = controller.Compute(actual_velocity, setpoint);
```

Where actual\_velocity is the vehicle's velocity and setpoint is the set distance (10m). The closer the velocity is to the setpoint, the smaller the control value (as less acceleration is required to reach the setpoint). 

If the set distance is violated, and the vehicle has not yet collided with the lead vehicle then the control value is computed with

```cpp
control = -controller.Compute(lead_distance-ego_distance, set_distance);
```

This value is negative to ensure lead\_distance-ego\_distance decelerated to reach the set distance value. If the vehicle seems to be on a collision course, the control value is computed with

```cpp
control = -controller.Compute(0, set_distance)
```

Only when the distance between the vehicles is less than 4m. This tricks the controller to thinking that the vehicle has collided without colliding yet. It might be a better idea to set it to some negative value to drive the controller to react harder.

Once the control value has been determined, for the ego vehicle the acceleration is determined by

```cpp
acceleration = 1/mass * (control - 0.5*rho*pow(velocity,2)*A*Cd - Crr*mass*g);
```

The bounds of the acceleration are checked, so cannot violate either the maximum or minimum. This was set to +3 and -3 $$ m/s^2 $$.

The velocity and position, once the acceleration has been determined, is set through

```cpp
velocity += acceleration * 0.2;
position += velocity * 0.2;
```

which increments with a dt of 0.2s.

This updates for every tick of the simulation. As the lead vehicle oscillates and causes the set distance to be violated, the ego vehicle responds by decelerating until the distance is no longer violated. It then acts to converge on the velocity setpoint.

## results

The PID values were P=80, I=10, D=10. The velocity and acceleration responses are below. The ego vehicle does not converge quickly, but it does converge to the velocity setpoint. The relationship between the acceleration and velocity values is correct, as observed by when the acceleration values are 0, the velocity values reach a maximum.

<p align="center"><img src="https://raw.githubusercontent.com/onlycase/adaptive-cruise-control/master/plots/vel-acc-responses.png"/></p>

Observing the distance traveled from the origin, the area between the curves at any instantaneous point signifies the difference between the vehicles. At the points where the curves seem to coincide, the distance was violated and the adaptive cruise controller responded. The second plot shows the relative distance oscillates as the lead vehicle tends to do so.

<p align="center"><img src="https://raw.githubusercontent.com/onlycase/adaptive-cruise-control/master/plots/distance.png"/></p>

The error values drive the PID output. Ideally the error converges to zero and the error accumulation reduces in growth over time. This is observed in the plots below.

<p align="center"><img src="https://raw.githubusercontent.com/onlycase/adaptive-cruise-control/master/plots/error.png"/></p>

## future

The simulation should be tested with more PID values to better refine the response. For the repo, see [here](https://github.com/onlycase/adaptive-cruise-control)
