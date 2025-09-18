---
Title: "Simulating Planetary Orbits Part 2: Programming the Numerical Integrator"
Date: 2025-09-15
---

{{< katex >}}
## Introduction
In [History of the *n*-Body Problem](https://ezminhas.github.io/posts/orbital-sim-pt1/), we discussed the development of the *n*-body problem into the exact sort of problem a computer simulation would excel at explaining. Now, it's time to build one. We'll be programming our *n*-body system-simulator in Python initially. Before beginning, it's important to plan out a tentative roadmap, to keep us on track.

### Roadmap
1. Understand the differential equations governing planetary motion.
2. Plan out the basic computational logic needed for our simulation, focusing on the Sun-Earth-Moon system at first.
3. Test the program with basic integration and vizualization. Does the simulation behave well?
4. Explore more optimal integration methods, and discuss the performance tradeoff. Achieve a simulation that predicts orbits well (what is a good prediction?).
5. Explore more robust vizualization methods, as well as graphical interactivity.

## The Problem

### Newton's Law of Universal Gravitation

Newton's law of universal gravitation, as formulated in many physics courses today, is given as this equation:
$$ F= G \frac{m_1 m_2}{r^2} $$
Where \(m_1, m_2\) are the masses of the two bodies, \(r\) is the separation between the two bodies, and \(G\) is the universal gravitation constant. In order to preserve the direction of the force, the law of universal gravitation in vectorial form is given as:
$$ \mathbf{F} = G \frac{m_1 m_2}{|\mathbf{r}_{21}|^3} \mathbf{r}_{21} $$

### Summing Forces and Presenting the Differential Equation
From here, we can pretty easily derive an equation for the acceleration of a body, influenced by however many massive bodies exist alongside it:
$$ \mathbf{a_i} = \sum_{i \neq j}^{N} G \frac{m_j}{|\mathbf{r_{ji}}|^3} \mathbf{r_{ji}} $$
Where \(\mathbf{r_{ji}} = \mathbf{r_j} - \mathbf{r_i}\)
But we're only really interested in position. Velocity and acceleration are just functions of position. So perhaps we should represent what we are looking for exactly. 
$$ \frac{d^2 \mathbf{r_i}}{d t^2} = \sum_{i \neq j}^{N} G \frac{m_j}{|\mathbf{r_{ji}}|^3} \mathbf{r_{ji}} $$

## The Solution

### Dealing with our Differential Equation (The First Attempt)

We now have a pretty innocuous-looking second-order ordinary differential equation (ODE) to solve. Except of course, there is no analytical solution our program can use. Instead, we will have to use numerical integration to find a solution.

Briefly though, what is numerical integration, and how can it help us? Numerical integration, or numerical methods for ordinary differential equations, are algorithms which can compute a numerical approximation of the solution of an otherwise unsolvable ODE.

#### Euler Method
The Euler method is perhaps the simplest numerical integration technique. Suppose we have some unknown curve modeled by a differential equation. We can write
$$ r'(t) = f (t,r) \quad r(t_0) = r_0 $$
The curve is unknown, so we don't know \(r(t)\). However, we do have our initial conditions, from which we can draw a tangent line to the curve beginning at point \(P_0 = (t_0, r_0)\). As long as our tangent line isn't too long, the point at which it ends, which we can call \(P_1\), can be treated as still on the curve. So using the coordinates of this new point, we can draw a new tangent line from \(P_1\) to \(P_2\), and so on.

So, how do we implement our *n*-body simulation in Python? Well, let's start by building a smaller problem, such as the Sun-Earth-Moon system. So, we have three bodies. What information do we need to keep track of? Well, firstly, we need some sort of data structure to store the masses of our objects. A simple 1-dimensional array should do. Next, we want to store the position, velocity and acceleration data for each of our bodies. Each of these is a vector describing \(x, y, z\) coordinate positions. To store everything, we want three \(3 \times 3\) matrices, which we can call \(R\) for position, \(V\) for velocity, and \(A\) for acceleration. We can also preset our position and velocity matrices with known data, while placing the Sun at the origin and treating it as a stationary object. For the three body problem we are dealing with right now, that's fine, but might need to be revisited as we simulate the entire solar system (the gas giants may slightly shift the center of mass of the solar system from the sun). Recall that the ODE being evaluated here includes \(G\), so declaring that here is helpful as well.

So, our program's declarations and initializations should look something like this:

```python

import numpy as np

# Set the number of bodies
N = 3

# Declare the individual masses of the bodies and store them in an array
m_1 = 1.989 * 10**30
m_2 = 5.972 * 10**24
m_3 = 7.348 * 10**22
masses = np.array([m_1, m_2, m_3])

# Set the gravitational constant
G = 6.674 * 10**-11

# Initialize position, velocity and acceleration arrays. Each array has shape (N, 3). The rows correspond to the bodies and the columns to the x, y, z coordinates.
R = np.array([[0.0, 0.0, 0.0],[1.5e11, 0.0, 0.0],[1.5e11 + 3.844e8, 0.0, 0.0]])
V = np.array([[0.0, 0.0, 0.0],[0.0, 29780.0, 0.0],[0.0, 29780.0 + 1022.0, 0.0]])
A = np.zeros((N, 3))

```

Euler's method can now be implemented. Firstly, we want to know the period of time over which we want to see the Earth and Moon orbits. I think 1 year is pretty reasonable. Another reasonable expectation is that the simulation will run and completely generate an animation within one minute. One year is \(86,400 \cdot 365\) seconds. If our step height is only one second, that's 31,536,000 steps. Clearly, our step height needs to be closer to one hour, or 3600 seconds. So let's set the number of steps, and step height.

```python
# Set the amount of time to stimulate, and the step size for Euler's method
t = 0;
# Step height, dt
dt = 3600
# The total number of steps is t_max / 8760
t_max = dt * 8760

```

Keep in mind that while the simulation could have been defined by the number of steps and step height, the variables were set up and named to more explicitly show the passage of time.

Now let's write out the portion of the program that will actually evaluate the ODE at each step:

```python
while(t < t_max):
    # Here we are calculating the right hand side of the ODE for each step, using the conditions given to us at that step
    for i in range(N):
        A[i] = np.array([0.0, 0.0, 0.0])
        for j in range(N):
            if(i != j):
                r_ji = R[j] - R[i]
                A[i] += G * masses[j] * r_ji / np.linalg.norm(r_ji)**3
    # Update time, positions and velocities using Euler's method
    t += dt
    R += V * dt
    V += A * dt

```

The entire block is within a `while` loop, to ensure the total number of steps are not exceeded. The `for` loop is a representation of the summation found within our ODE, which we earlier derived directly from Newton's law of universal gravitation. The final two lines are where the Euler method is actually being implemented, giving us the vital position and velocity data needed.

Now, we need something to visualize the simulation. A library like `matplotlib.animation` is perfect for doing this in Python. In the future, we might explore other tools like `makie.jl` in Julia, and more, but for now, this will do. The very first thing we need is a representation of every computed position throughout the entire simulation. Obviously, `R` is being updated every step, so let's add an array called `R_history` that will store our entire position history. Adding this storage structure is quite simple, and once it's done, our program looks like this:

```python
# Initializations and declarations here

# Create an array to store the history of positions for plotting later
R_history = []

# Main loop using Euler's method to update positions and velocities
while(t < t_max):
    # Here we are calculating the right hand side of the ODE for each step, using the conditions given to us at that step
    for i in range(N):
        A[i] = np.array([0.0, 0.0, 0.0])
        for j in range(N):
            if(i != j):
                r_ji = R[j] - R[i]
                A[i] += G * masses[j] * r_ji / np.linalg.norm(r_ji)**3
    # Update time, positions and velocities using Euler's method
    t += dt
    R += V * dt
    V += A * dt
    R_history.append(R.copy())

# Convert R_history to a numpy array for easier manipulation later
R_history = np.array(R_history)
print(R_history)

```

We can now use `matplotlib.animation` to generate a visualization of our simulation.

As an example, let's use one of simplest ODEs and its solution, setting the initial condition \(t(0) = 0, x(t_0) = 1\),

$$ x'(t) = x $$

and its analytical solution,

$$ x(t) = e^{t} $$

Let's Euler's method to evaluate this ODE for \(0 \leq t \leq 10\). Performing 1000 steps with a step height of 0.01 per step seems appropriate for this time range. So, what does it look like?

{{< figure
    src="fig1.png"
    >}}

Looks pretty good, right? Sure, there's some deviation past about \(t = 9\) or so, but it seems like this has a lot of application. However, our domain is pretty small. Suppose \(t\) is in units of seconds. Then this plot shows us 10 seconds of growth. What if we want to figure out 24 hours of growth? That's 86,400 seconds. If we want to keep our step height of 0.01, we need 8.64 million steps. Sounds fine, but what if we need a month of growth? A year? That number of steps keeps growing bigger. So for now, let's say that we have 100,000 computation tokens, and each time we run our program, every step computed costs a token. So, to see our growth for 24 hours, while making sure we don't use more tokens than we have, we need to set step height to \(h = 0.864\). What kind of effect does this higher step height have even in the first 10 seconds?

{{< figure
    src="fig2.png"
    >}}



Recall our ODE for the *n*-body problem,
$$ \mathbf{r}''(t) = \sum_{i \neq j}^{N} G \frac{m_j}{|\mathbf{r_{ji}}|^3} \mathbf{r_{ji}}  $$
