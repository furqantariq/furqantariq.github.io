---
title: "Implementing Raycaster Engine"
date: 2023-05-21T15:47:30+02:00
draft: true
---

## Introduction

Raycasting is a technique in Computer Graphics to create three-dimensional (3D) perspective image from two-dimensional 
(2D) data. Back in the days, when game development was in its nascency and computer were slow with limited memory, 
it was one of the efficient ways to create scenes from a player's perspective. It gained prominence in games like 
Wolfenstein 3D and Doom which also happened to introduce the concept of first-person-shooter (FPS) games.


{{< youtube 561sPCk6ByE >}}


## Raycasting Algorithm

### Data Representation

Before delving into implementation of raycasting, we have to define our game world data in 2D-matrix in which 
each element corresponds to square cell in the game world with a height and width. The value "**1**" represents that
this cell is a wall and "**0**" represents an empty space where player can move. In the figure below, the player $P$ is 
standing at $x=2.5, y=0.5$ position and he is looking toward wall with an direction angle $P_\theta$ of 90 degrees. The
fraction part of the position indicates that player is standing in the middle of cell. 
The player also has a Field of View (FOV) angle that represents the total observable area that can be seen at once by 
player's eye.  

{{<math>}}
$$
    world (x, y) = 
    \begin{bmatrix}
        0 & 0 & 1 & 0 & 0 \\ 
        1 & 1 & 0 & 1 & 1 \\
        0 & 0 & 0 & 0 & 0 \\
        0 & 0 & 0 & 0 & 0 \\
        0 & 0 & 0 & 0 & 0 \\
    \end{bmatrix}
$$
$$ FOV = 70^\circ $$
$$ P_{x,y} = (2.5, 0.5), P_\theta = 90^\circ $$

{{</math>}}

{{<figure src="map.svg" width="50%" height="50%" >}}

### Casting Rays
In principle, raycasting works by projecting rays from players position across his field of view and tracing them to 
the point where they intersect with wall, object or end of map. The number of rays are casted are equal to the width 
of the projected screen because each ray is responsible for drawing to its respective column of pixels in the projected screen. 
After finding intersection points, the distance between the player position and each intersection point is calculated and 
based on that distance object or wall is drawn into respective column. If distance is greater the object appears smaller or bigger otherwise.

{{<figure src="rays.svg" width="100%" height="100%" >}}
{{<figure src="perspective.svg" width="100%" height="100%" >}}


### Tracing Rays

Eeach ray $R_i$ originates from the player position $P_{x,y}$ with an angle $\theta_i$ and before casting rays, 
their angles must be determined first. Since we know that number of rays are equal to the screen width, 
then the angle between two consective rays will be $ \frac{FOV}{screen\text{-}width} $ and left most ray $R_0$ will have an angle
$ \theta_0 = P_\theta + \frac{FOV}{2} $ so the angle of $i$th ray would be

$$ \theta_i = (P_\theta + \frac{FOV}{2}) - i\cdot(\frac{FOV}{screen\text{-}width}) $$


Now the tracing of rays involves finding all the horinzontal and vertical grid lines of the map that particular ray $R_i$
falls on. On each horizontal grid line we check whether next horizontal cell infront of ray is wall or not and on 
each vertical grid line we check the same but with vertical cells. If we encounter any cell or wall then 
we stop at that point and that point is our **intersection point** $V_{i_{x,y}}$ or $H_{i_{x,y}}$. During tracing, If we dont encounter any object and 
reach end of the map then we consider our distance to be infinite.


Since our player $P_{x,y}$ standing somewhere inside the cell, so we have to find the nearby horizontal $h_{i_{x,y}}$ and 
vertical $v_{i_{x,y}}$ grid line and that we can find by taking difference between $\text{cell size}$ and player position 
$P_{xy}$, depending on the direction of the ray $R_i$ (i.e. we have to move backward if $\theta_i \ge 180^\circ$). 
The directional part can easily be fixed by just multiplying it with sign value of $\cos\theta_i$ (in case of horizontal) and $\sin\theta_i$ (in case of vertical).

The corresponding component can be calculated by the highschool fact that $ \tan\theta = \frac{\text{perpendicular}}{\text{base}} $.

{{<math>}}
$$ 
\begin{align}
h_{i_x} &= \frac{\cos\theta_i}{|\cos\theta_i|}\cdot(cellsize - P_x) \\[2ex] 
h_{i_y} &= h_{i_x}\cdot\tan\theta_i \\ 
\end{align}
$$
{{</math>}}

similarly,

{{<math>}}
$$
\begin{align}
v_{i_y} &= \frac{\sin\theta_i}{|\sin\theta_i|}\cdot(cellsize-P_y)\\[2ex] 
v_{i_x} &= \frac{v_{i_y}}{\tan\theta_i} \\
\end{align}
$$
{{</math>}}

After finding initial gridline points, now its time to move to the next gridline points along the ray $R_i$.
For this, we can a define a recursive function $walk(x,y)$ that jumps to next gridline point by adding $cellsize$
to the respective component and stops when it encounters an object.


{{<math>}}
$$
walk\text{-}h(x,y) = 
\begin{cases}
(x, y), & \text{if $world( \lfloor\frac{x}{cellsize}\rfloor + \frac{\cos\theta_i}{|\cos\theta_i|}, \lfloor\frac{y}{cellsize}\rfloor) = 1$} \\[2ex]
walk\text{-}h(\underbrace{x + \frac{\cos\theta_i}{|\cos\theta_i|}\cdot cellsize}_\acute{x}, \acute{x}\cdot\tan\theta_i), & \text{else} \\
\end{cases}
$$
{{</math>}}

similarly,

{{<math>}}
$$
walk\text{-}v(x,y) = 
\begin{cases}
(x, y), &\text{if $world( \lfloor\frac{x}{\text{cell size}}\rfloor, \lfloor\frac{y}{cell size}\rfloor  + \frac{\sin\theta_i}{|\sin\theta_i|}),= 1$} \\[2ex]
walk\text{-}v(\frac{\acute{y}}{\tan{\theta_i}}, \underbrace{y + \frac{\sin\theta_i}{|\sin\theta_i|}\cdot cellsize}_{\acute{y}}), & \text{else} \\
\end{cases}
$$
{{</math>}}

Now our intersection point will be sum of distance to nearby gridline point and walking further alongside ray, until it encounters object.

$$ H_{i_{x,y}} = h_{i_{x,y}} + walk\text{-}h(h_{i_{x,y}}) $$
$$ V_{i_{x,y}} = v_{i_{x,y}} + walk\text{-}v(v_{i_{x,y}}) $$



{{<figure src="raytracing.svg" width="100%" height="100%" >}}


After this we will have two **intersection points** on which ray is intersecting both vertically and horizontally 
with a cell. Now we need to find distance between the player position and nearest **intersection point**.
That can be done by taking euclidean distance between these two **intersection points** and the minimum of these two 
distances will be our distance to nearest interseciton point. 

$$ \acute{d_i} = \min(\sqrt{ (P_x - H_{i_x} )^2 - (P_y - H_{i_y})^2} ,  \sqrt{ (P_x - V_{i_x} )^2 - (P_y - V_{i_y})^2}) $$


### Fixing Fisheye-Lens

Although we have distance now but the distance is not correct and it is actually slightly distorted. If we use this
distance in drawing our scene, It will give an effect known as "fishbowl effect". The reason behind is because rays
that are away from the player direction are actually longer then what their actual length is, as compare to the rays 
that are closer to player direction and these longer rays give an illusion that object is farther away, hence the fishbowl
effect. To fix that is quite simple, all we have to do is multiply distorted distance with sine of abosulte difference of
ray angle and player direction.  

{{<figure src="fishbowl-effect.svg" height="100%" width="100%">}}

$$ d_i = \acute{d_i} \cdot \cos(|\theta_i-P_\theta|) $$

### Projected Height

The purpose of finding distance to object was to find out the height of object in projected screen. We have already
established earlier that everything player see is of same height (i.e. `cell_height`). Now if we look at the figure,
the actual object lies at distance $\overline{OF}$ from player $P$ with height $\overline{AB}$ and line $\overline{XY}$ 
represents the projected height of the object in projection plane/screen, that also lies at certain distance $\overline{OD}$ 
from player $P$ in same direction. This gives us two similar triangles $ \triangle AOB $ and $ \triangle XOY $ with 
equal angles that means the ratios of their corresponding sides are equal.

{{<figure src="triangles.svg" height="100%" width="100%">}}

$$ \triangle AOB \sim \triangle XOY$$

$$ \frac{\overline{XY}}{\overline{AB}} = \frac{\overline{DO}}{\overline{FO}} $$

Since $\overline{AB}$ is `cell_height`, $\overline{FO}$ is the distance we calculated earler and $\overline{OD}$ is 
upto us to decide how much away projection plane/screen should be. For the sake of simplicity, we can take it 1 unit. 
That leave us $\overline{XY}$ which is our projected height. 

$$ h_i = \frac{cellsize}{d_i} $$

### Rendering Scene

By casting a ray from player we finally able to get projected height of an object it intersecting with or zero height 
if it doesn't intersect with anything. As it has been said earlier, that each ray is responsible for drawing to 
corresponding column of pixels in projection screen. That means if we cast all the rays from players, that will give us 
projected heights for all columns of pixels. Now, to render our final scene, all we have to do is in each pixel column
at the center we fill those projected height pixels with object color. 

{{<figure src="rendering.svg" height="100%" width="100%">}}

# DEMO

{{< jsfiddle "furqantariq/364w95pt" >}}

*This blogpost will be divided into two parts and in next part i will write about applying textures and implementing 
weather effects (fog, night and day) in raycasting*
