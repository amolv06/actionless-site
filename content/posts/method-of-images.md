+++
title = "The Method of Images"
date = 2024-09-08T00:00:00-05:00
tags = ["physics"]
draft = false
+++

## Introduction {#introduction}

The method of images is almost a real life mathematical cheat code. Imagine being able to solve what happens due to some intricate arrangement of charges on a conducting surface, but pretending as though it's a single point charge. When I first saw this, I was quite stunned. It seems absurd that to solve one problem you solve, more or less, a completely separate problem. However the clever application of a mathematical principle ensures that the solutions to the two problems are the same. Instead of solving a hard physics problem it allows you to solve a much easier, barely related problem, and then insist that the solution to the two is the same.


## The Method of Images Without the Math {#the-method-of-images-without-the-math}

Imagine placing two electrons near each other -- they would repel one another. This happens because both electrons carry an **electric charge**, and charged particles exert a force on each other. Electric charge comes in two varieties: negative and positive. Like charges repel, while opposites attract.

We can think of a charged particle as creating a force field that permeates all space, called the **electric field**, which acts on other charged particles. When a secondary charge is placed within this field, it reacts accordingly -- repelling if the charges are the same and attracting if they are opposite. But when analyzing the second particle's behavior, we don't need to know anything about how the electric field was created. Everything we need to understand the particle's behavior is contained in the electric field itself. This allows us to focus solely on the electric field and the second particle, ignoring the first particle entirely.


## A Brief Review {#a-brief-review}

This post assumes you're familiar with basic vector calculus and have taken an introductory course in electromagnetism, however I'll still review some foundational concepts. I'll introduce these concepts as fact, without delving into their justification, though they are theoretically and experimentally well established.


### Coulomb's Law {#coulomb-s-law}

We begin by imagining point particles that possess a property known as electric charge. The value of this electric charge can range from \\((-\infty, \infty)\\), and it is typically measured in Coulombs (C). In electrostatics the elementary unit of charge has a charge of \\(1.602 \times 10^{-19}\textrm{C}\\). This is the magnitude of the charge of the electron. **Coulomb's Law** states that if the two particles have charge of \\(q\_1\\) and \\(q\_2\\), and are separated by a distance of \\(r\\), then the force between them is

\\[\vec{F} = \frac{q\_1 q\_2}{4 \pi \epsilon\_0 r^2} \hat{r}.\\]

Note that the arrow above the F and the hat above the r indicates this is a vector equation. The force has both a magnitude and a direction: if both charges have the same sign then the force on each is repulsive, meaning that the charges will be pushed away from each other. If the charges have opposite signs the force is attractive.

You might also notice the factor \\(\frac{1}{4 \pi \epsilon\_0}\\). Its form is the result of historical convention, but numerically it is approximately \\(8.988 \times 10^9 \\; \textrm{N} \cdot \textrm{m}^2 \cdot \textrm{C}^{-2}\\). A little dimensional analysis shows that if the charges are given in Coulomb's and distances in meters then applying Coulombs law will give us forces in the familiar unit of Newtons.


### The Electric Field {#the-electric-field}

Based on Coulomb's law we can imagine that each charge induces a force-field permeating all of space. If we

<center>

<script type="text/tikz">
  \begin{tikzpicture}
\draw[fill=gray!20] (-8,-0.5) rectangle (8,0.5);
   \node at (0,0.75) {Conducting Plate};

% Positive Charges on one side of the plate
\foreach \x in {-2.5,-1.5,-0.5,0.5,1.5,2.5} {
    \node[scale=1.5, red] at (\x, 0.3) {$+$};
}

% Negative Charges on the opposite side of the plate
\foreach \x in {-2.5,-1.5,-0.5,0.5,1.5,2.5} {
    \node[scale=1.5, blue] at (\x, -0.3) {$-$};
}

% Electric Field lines (E field pointing left to right)
\foreach \x in {-2, -1.5, -1, 1, 1.5, 2} {
    \draw[->, thick] (\x, -4) -- (\x, 4) node[pos=0.9, right] {$\vec{E}$};
}

% Label of E field
\node at (4, 2) {$\vec{E}$ (External Electric Field)};
\end{tikzpicture}
</script>

</center>

<script type="text/tikz">
  \begin{tikzpicture}
\draw (0,200) circle (1in);
\line
\end{tikzpicture}
</script>
