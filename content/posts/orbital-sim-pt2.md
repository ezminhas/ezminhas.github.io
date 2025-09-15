---
Title: Simulating Planetary Orbits Part 2: Programming the Numerical Integrator
Date: 2025-09-15
---
{{ <katex> }}

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
$$ F = G \frac{m_1 m_2}{r^2} $$
Where \(m_1, m_2\) are the masses of the two bodies, \(r\) is the separation between the two bodies, and \(G\) is the universal gravitation constant. In order to preserve the direction of the force, the law of universal gravitation in vectorial form is given as:
$$ \mathbf{F} $$
