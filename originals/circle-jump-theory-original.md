# "Circle-Jump Theory" by injx

This article will look at the mathematics of circle-jumping in order solve some useful problems. Much of the content of this article is a natural progression from [Strafing Theory](/originals/strafing-theory-original.md) and so to avoid repeating things from that article, I recommend you attempt to read Strafing Theory first.

A circle-jump is a very useful technique used to gain **initial speed**. It allows a player to jump across large gaps and is used as a precursor to strafe-jumping and bunny-hopping. It is so-called as it requires the player to run along the ground following the **path of a small circle** before jumping. As we will see it is this circular path that causes an increased ground speed.

There are two parts to a circle-jump, the **ground** part and the **air** part. First let's look at the ground part.

There are two differences between being on the ground and being in the air.

The first difference is simple: the ground acceleration is much greater than the air acceleration. The acceleration is defined as ![a=AsT](/assets/circle-jump-theory/math-accel.png), where _T_ is the **frame-time**, _s_ is the **speed limit** (g_speed) and _A_ is some constant depending on where the player is (ground/air/water etc.). In Strafing Theory we learnt that for air acceleration, _A_\=1. However here we are dealing with **ground acceleration**, so we have _A_\=10 for VQ3 and _A_\=15 for CPM.

The second difference is the friction, and it is this friction that causes the **maximum attainable ground speed to be limited**, so it's important that we understand how it works.

In the Q3 engine, friction is implemented by decreasing the velocity every frame. The amount that the velocity is decreased by is dependent on the size of the velocity itself, which is the same way that friction in the real world usually works. When the velocity is low, the players applied acceleration is enough to overcome the friction and result in an increase in speed, however as the player's speed grows, the friction becomes larger, while the player's acceleration, _a_, is a constant. This means that at some point the friction will balance out the player's own acceleration and the speed will not increase any further. Thus we can expect to find a _terminal velocity_.

Since the friction is dependent on the player's speed, it would normally take a long time for the friction to bring the player to a complete stop when no acceleration is being applied (low speed -> low friction -> low deceleration). To combat this, the friction has a lower limit. When the player's speed is below 100ups, the applied friction is the same as it is at 100ups.

![friction definition](/assets/circle-jump-theory/math-frictiondef.png)

Here, _d_ is the **100ups limit**, and _c_ is the **friction factor**, which is multiplied by the speed, _v_, and the **frame-time**, _T_, to give the **decrease in speed per frame**. For both VQ3 and CPM, _c_\=6.

So apart from these two differences, the mechanics of moving on the ground is exactly the same as in the air. Therefore we can use many of the results from Strafing Theory and apply them to this problem by making the following replacement

![replacement](/assets/circle-jump-theory/math-replacement.png),

and by remembering that _a_ is now the **ground** acceleration.

![angle delta](/assets/circle-jump-theory/delta.png)

In Strafing Theory I stated that the optimal acceleration angle, delta _δ_, is the 'smallest angle at which we receive the full acceleration'. This statement holds, but we have to be careful when applying to small velocities. When our speed is below (_s_ – _a_), the optimal angle is always 0degrees (acceleration in the same direction as the velocity). Above (_s_ – _a_), the optimal angle delta is given by the same expression from Strafing Theory (after the replacement above has been made),

![optimal angle delta](/assets/circle-jump-theory/math-deltaopt.png).

What this means is that you begin stationary and start by accelerating in a straight line, in the same direction as your velocity. This is because it is the quickest way to accelerate up to the speed limit enforced by g_speed. Only once the speed limit starts to affect your attempt to accelerate further do you need to apply accelerations at angles to your velocity, and hence begin to turn.

Once the need to turn becomes necessary, applying an optimal angle gives a maximum resultant speed of

![max resultant speed](/assets/circle-jump-theory/math-resultantopt.png),

and from this, it's easy to find the expression for the **maximum ground speed**,

![max speed](/assets/circle-jump-theory/math-vmax.png),

which gives us values of **410ups for VQ3** and **497ups for CPM**, at 125fps. Note that if there was no friction (_c_ = 0), there would in principle be no limit on the maximum speed. This would be an example of being on an **ice brush**, or indeed in the air. However there are inevitably other aspects of the engine that put limits on very high speeds.

It's interesting to note that the maximum ground speed is dependent on the frame-time, _T_, and hence the framerate at which the physics is being calculated. A quick graph shows us how the max ground speed varies with framerate.

![max resultant speed](/assets/circle-jump-theory/graph-vmax.png)

You can see that this effect is only really noticeable at very low framerates. This effect is just due to the discrete frame-based nature of the engine, and although (like so many other aspects) it isn't explicitly coded into the physics, it is intrinsic to the engine.

As the framerate increases, the value of v-max tends to

![vmax=s*sqrt(A/c)](/assets/circle-jump-theory/math-vmaxinf.png).

This result is useful, for example if a developer wanted to increase ground friction without affecting circle-jumping, they would need to increase ground acceleration by the same proportion. Alternatively the max ground speed expression could be used to maintain v-max for a specific framerate.

So now we know about the maximum ground speed, let's look at how to achieve it. When the optimal angle delta is applied, the **change in direction per frame** of the velocity is given by

![change in direction per frame](/assets/circle-jump-theory/math-gphione.png).

We can use this along with the other results above to graph the **rate at which we need to turn** our view in order to gain the **maximum ground speed**.

![perfect cj](/assets/circle-jump-theory/graph-cj.png)

As the graph shows, the turning rate begins high but quickly reduces to a **minimum value** as the velocity tends to its maximum. The value of this minimum turning rate is given by

![min turning rate](/assets/circle-jump-theory/math-omegamin.png),

which gives **294 degrees/sec for VQ3** and **445 degrees/sec for CPM**.

We can also look at the path that a circle-jump follows along the ground to see how circular it actually is.

![ground path](/assets/circle-jump-theory/graph-groundpathvq3.png). [Click for CPM graph](/assets/circle-jump-theory/graph-groundpath.png).

Here the red line shows the initial straight-line acceleration, and the blue curve shows the turn. The blue curve is indeed very circular: if the blue curve was extended it would circle around and roughly join its own starting point. The orange arrow is the general direction in which we are trying to move in.

Armed with this information, we can calculate the radius of the circle in which we must move.

![R=vmax/omegamin](/assets/circle-jump-theory/math-radius.png) \[omega must be in **radians** for this one.\]

The circular path has a **radius of 80 units for VQ3**, and a **radius of 64 units for CPM**. This example ground path takes about **two-thirds of a second** to complete for VQ3 and just under **half a second** to complete for CPM, from start to the take-off point.

The starting angle is specially chosen to give an optimal take-off angle. The optimal take-off angle itself is determined by the 'air' part which will come later. In these examples the starting angles are 150 and 160 degrees for VQ3 and CPM respectively.

Now let's consider the air part. Since the player is airborne, there is no friction, and the results from Strafing Theory can be applied directly. However in Strafing Theory we were dealing with high speeds which meant we could use a constant angle over the period of each jump, and only had to increase the angle at air-changes.

Because our circle-jump is involving lower speeds, the applied acceleration needs to be constantly adjusted to maintain an optimal effect. Because a circle-jump is traditionally performed by using both the Forwards and Strafe keys simultaneously, the **air acceleration** method is the same for both VQ3 and CPM, i.e. standard VQ3 strafing with **_A_\=1**.

Using the results from Strafing Theory, a numerical analysis can determine the take-off and landing angles required to give the greatest distance travelled in the direction we generally wish to travel in. However, we can still use analytical methods to gain a good approximation of these values.

We know from Strafing Theory that the when applying optimal air acceleration angles, the increase in speed per frame decreases as we speed up, therefore we experience the greatest increase in speed immediately after take-off. The change in direction per frame however has a peak in the middle of the air part. We can find when this peak occurs, and the value that it gives, by taking the expression for the change in direction per frame,

![change in direction per frame](/assets/circle-jump-theory/math-phi1.png),

and using calculus, which gives a peak change in direction of

![peak change in direction per frame](/assets/circle-jump-theory/math-phi1max.png),

occurring at

![speed at phi1max](/assets/circle-jump-theory/math-vphi.png).

So firstly we can use our initial increase in speed, from take-off, to produce an **upper bound** for the landing speed.

![landing speed upper bound](/assets/circle-jump-theory/math-vub.png) (where _v_<sub>0</sub> is the take-off speed)

This gives values of **585ups for VQ3** and **641ups for CPM**.

Similarly, we can use the peak change in direction per frame to produce an **upper bound** for the total change in direction caused by the air part of the circle-jump.

![total change in direction upper bound](/assets/circle-jump-theory/math-phiub.png), which gives 20 degrees.

![air path](/assets/circle-jump-theory/phi.png)

In order to travel the greatest distance in a chosen direction of overall motion, the path needs to be orientated appropriately. A good approximation to the optimum orientation is when the angle between the velocity and direction of overall motion is equal and opposite for take-off and landing.

By setting up the path in this way, we can determine values for the take-off and landing **view**\-angles. Let angle epsilon, _ε_, be the view-angle relative to the direction of overall motion. At **take-off** this view-angle is

![epsilon 0](/assets/circle-jump-theory/math-epsilon0.png), which gives **16 degrees for VQ3** and **5 degrees for CPM**.

On **landing** from the circle-jump, the view-angle is

![epsilon N](/assets/circle-jump-theory/math-epsilonn.png), which gives **\-22 degrees for VQ3** and **\-25 degrees for CPM**.

The negative landing angle indicates that it is on the opposite side to the starting angle. A quick numerical analysis shows that these results are quite accurate despite the two approximations used. The take-off angles agree to the nearest degree, and the landing angles are one or two degrees smaller in magnitude in the numerical analysis.

From this, the distance travelled in the desired direction is approximately given by

![distance](/assets/circle-jump-theory/math-distance.png),

giving 350 units for VQ3 and 400 units for CPM.

However if you include stepping, then there are about 96 air-frames available (at 125fps), rather than the usual 88 frames, and so using _N_\=96, the distances become 388 and 442 units respectively.

Note that there are some little quirks in the Q3 engine that affect movement, one noticeable example being the rounding of the velocity components at each frame. In general these additional effects statistically cancel out over time when speaking of ground- and air-acceleration, and can only be taken advantage of by a very specific setup and precise user actions - a bot or script. Either way, this article is concerned with the concept of circle-jumping itself, and not additional quirks which may be specific to the Q3 engine, in the same way that Strafing Theory is concerned purely with the concept of strafe-jumping itself.
