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
$$ r'(t) = f (t,r) \\quad r(t_0) = r_0 $$
The curve is unknown, so we don't know \(r(t)\). However, we do have our initial conditions, from which we can draw a tangent line to the curve beginning at point \(P_0 = (t_0, r_0)\). As long as our tangent line isn't too long, the point at which it ends, which we can call \(P_1\), can be treated as still on the curve. So using the coordinates of this new point, we can draw a new tangent line from \(P_1\) to \(P_2\), and so on.

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
