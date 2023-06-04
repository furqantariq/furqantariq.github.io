---
title: "Raycasting - part 1"
date: 2023-05-21T15:47:30+02:00
tags: ["computer-graphics"]
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
each element corresponds to square cell in the game world with a height, width and breadth of $\textit{cellsize}$. 
The value "**1**" represents that this cell is a wall and "**0**" represents an empty space where player can move. 
In the figure below, the player $P$ is standing at $x=2.5, y=0.5$ position and he is looking toward wall with an 
direction angle $P_\theta$ of 90 degrees. The fraction part of the position indicates that player is standing in the middle of cell. 
The player also has a Field of View (FOV) angle that represents the total observable area that can be seen at once by 
player's eye.  

{{<math>}}
$$
    map (x, y) = 
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

{{<figure src="map.svg" width="65%" height="65%" >}}

### Casting Rays
In principle, raycasting works by projecting rays from players position across his field of view and tracing them to 
the point where they intersect with wall, object or nothing. The number of rays that are casted are equal to the width 
of the screen because each ray is responsible for drawing to its respective column of pixels in the projected screen. 
After finding intersection points, the distance between the player position and each intersection point is calculated and 
based on that distance object or wall is drawn into respective column. Objects that are closer to the camera will appear larger, 
while those farther away will appear smaller, simulating depth perception.

{{<figure src="rays.svg" width="100%" height="100%" >}}
{{<figure src="perspective.svg" width="100%" height="100%" >}}


### Ray-Object intersection

Each ray $R_i$ originates from the player position $P_{x,y}$ with an angle $\theta_i$ and before casting rays, 
their angles must be determined first. Since we know that number of rays are equal to the screen width, 
then the angle between two consective rays will be $ \frac{FOV}{\textit{screen-width}} $ and left most ray $R_0$ will have an angle
$ \theta_0 = P_\theta + \frac{FOV}{2} $ so the angle of $i$th ray would be

$$ \theta_i = (P_\theta + \frac{FOV}{2}) - i\cdot(\frac{FOV}{\textit{screen-width}}) $$


The tracing of rays involves finding all the horizontal and vertical gridlines of the map that particular ray $R_i$
falls on. On each horizontal gridline we check whether next horizontal cell infront of ray is wall or not and on 
each vertical gridline we check the same but with vertical cells. If we encounter any cell or wall then 
we stop at that point and that point is our intersection point $V_{i_{x,y}}$ or $H_{i_{x,y}}$ for ray $R_i$. During tracing, If we dont encounter any object and 
reach end of the map then we consider our distance to be infinite.


Since player $P_{x,y}$ standing somewhere inside the cell, so we have to find the nearby horizontal $h_{i_{x,y}}$ and 
vertical $v_{i_{x,y}}$ gridline and that we can find by taking difference between $\textit{cellsize}$ and player position 
$P_{x,y}$, depending on the direction of the ray $R_i$ (i.e. we have to move backward if $\theta_i \ge 180^\circ$). 
The directional part can easily be fixed by just multiplying it with sign value of $\cos\theta_i$ (in case of horizontal) and $\sin\theta_i$ (in case of vertical).

The corresponding component can be calculated by the trigonometric ratio $ \tan\theta = \frac{\text{perpendicular}}{\text{base}} $.

{{<math>}}
$$ 
\begin{align}
h_x &= \frac{\cos\theta_i}{|\cos\theta_i|}\cdot(cellsize - P_x) \\[2ex] 
h_y &= h_x\cdot\tan\theta_i \\ 
\end{align}
$$
{{</math>}}

similarly for vertical gridlines,

{{<math>}}
$$
\begin{align}
v_y &= \frac{\sin\theta_i}{|\sin\theta_i|}\cdot(cellsize-P_y)\\[2ex] 
v_x &= \frac{v_y}{\tan\theta_i} \\
\end{align}
$$
{{</math>}}

After finding initial gridline points, now its time to move to the next gridline points along the ray $R_i$.
For this, we can a define a recursive function $\textit{next-grid}(x,y)$ that jumps to next gridline point by adding $\textit{cellsize}$
to the respective component and stops when it encounters an object.


{{<math>}}
$$
\textit{next-grid-h} (x, y) = 
\begin{cases}
(x, y), & \textit{if map$( \lfloor\frac{x}{cellsize}\rfloor + \frac{\cos\theta_i}{|\cos\theta_i|}, \lfloor\frac{y}{cellsize}\rfloor) = 1$} \\[2ex]
\textit{next-grid-h}(\underbrace{x + \frac{\cos\theta_i}{|\cos\theta_i|}\cdot cellsize}_\acute{x}, \acute{x}\cdot\tan\theta_i), & \text{else} \\
\end{cases}
$$
{{</math>}}

similarly,

{{<math>}}
$$
\textit{next-grid-v}(x,y) = 
\begin{cases}
(x, y), &\textit{if map$( \lfloor\frac{x}{cellsize}\rfloor, \lfloor\frac{y}{cell size}\rfloor  + \frac{\sin\theta_i}{|\sin\theta_i|}),= 1$} \\[2ex]
\textit{next-grid-v}(\frac{\acute{y}}{\tan{\theta_i}}, \underbrace{y + \frac{\sin\theta_i}{|\sin\theta_i|}\cdot cellsize}_{\acute{y}}), & \textit{else} \\
\end{cases}
$$
{{</math>}}

The pictorial form of ray-object intersection can also be seen below.

{{<figure src="raytracing.svg" height="100%" width="100%">}}


So the intersection points $H_{i_{x,y}}$ and $V_{i_{x,y}}$ that intersects ray will be 

$$ H_{i_{x,y}} = \textit{next-grid-h}(h_{i_{x,y}}) $$
$$ V_{i_{x,y}} = \textit{next-grid-v}(v_{i_{x,y}}) $$

where $h_{i_{x,y}}$ and $v_{i_{x,y}}$ are nearby gridline points for any $R_i$ 


Between these two intersection points, only the nearest point will be used in scene rendering because the farthest point
will always be hidden behind. The intersection point itself has no value for us unless we are applying textures, 
but we are interested in the distance to nearest intersection point. This can be found by taking minimum of the 
Euclidean distances from player position $P_{x,y}$ to both of these points $H_{x,y}$ and $V_{x,y}$

$$ \acute{d_i} = \min(\sqrt{ (P_x - H_{i_x} )^2 + (P_y - H_{i_y})^2} ,  \sqrt{ (P_x - V_{i_x} )^2 + (P_y - V_{i_y})^2}) $$


### Fixing Fisheye-Lens

Although we have distance now but the distance is not correct and it is actually slightly distorted. If we use this
distance in drawing our scene, It will give an effect known as "fisheye effect". The reason behind is because the rays
that are away from the player direction are actually longer in length then the rays that are parallel to player direction
and these longer rays give an illusion that object is farther away, hence the fisheye
effect. To fix that is quite simple, all we have to do is multiply distorted distance with cosine of abosulte difference of
ray angle $\theta_i$ and player direction $P_\theta$.  

{{<figure src="fishbowl-effect.svg" height="100%" width="100%">}}

$$ d_i = \acute{d_i} \cdot \cos(|\theta_i-P_\theta|) $$

### Projection

The purpose of finding distance to object was to find out the projected height of the object. We have already
established earlier that everything player see is of same height. Now if we look at the figure below,
the actual object lies at distance $\overline{OF}$ away from player $P$ with height $\overline{AB}$ and the line $\overline{XY}$ 
represents the projected height of the object in projection screen, that also lies at certain distance $\overline{OD}$ 
away from player $P$ in same direction. This gives us two similar triangles $ \triangle AOB $ and $ \triangle XOY $ with 
equal angles that means the ratios of their corresponding sides are equal.

{{<figure src="triangles.svg" height="100%" width="100%">}}

$$ \triangle AOB \sim \triangle XOY$$

$$ \frac{\overline{XY}}{\overline{AB}} = \frac{\overline{DO}}{\overline{FO}} $$

Since $\overline{AB}$ is $\textit{cellsize}$, $\overline{FO}$ is the distance we calculated earler and $\overline{OD}$ is 
upto us to decide how much away projection screen from player $P$ should be, that leave us $\overline{XY}$ which is 
the projected height $h_i$ of the object with which the ray $R_i$ intersects. 

$$ h_i = \frac{cellsize}{d_i} \cdot \textit{projection-distance} $$

### Rendering Scene

To render final scene, raycasting uses column-based rendering. In which we iterates through each column $c_i$ in screen 
and at the center of the pixel column, we fills all corresponding pixels with color of object that intersects with ray $R_i$ 
at the calculated projected height $h_i$. In each iteration, rest of the pixels are filled with environment color
e.g. sky, floor ... etc.

{{<figure src="rendering.svg" height="100%" width="100%">}}

In the end, after rendering process, we will have a 2D image that represents a 3D scene with illusion of depth and 
object visibility from 2D data, the demonstration of which can be seen below.


# DEMO

Use **W**,**A**,**S** and **D** keys to update $P_{x,y}$ and modify $P_\theta$ with mouse pointer.

Readers are encouraged to open 'Typescript' tab and to experiment with different colors, `WORLD_MAP` variable and various
other configurable variables.

{{<codepen src="https://codepen.io/furqant/embed/XWPRyYv?default-tab=result&editable=true&theme-id=light" height="570">}}

*This blogpost will be divided into two parts and in next part i will write about applying textures and implementing 
weather effects (fog, night and day) using raycasting*
