---
layout: post
title:  "Notes on Quadratic Curves in ℝ²"
date:   2015-01-15 14:00:00
categories: graphics
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script>
MathJax.Hub.Config({
    jax: ["input/TeX","output/HTML-CSS"],
    displayAlign: "left",
    displayIndent: "2em",
});
</script>

Quadratic Curves
================

The <a href="http://mathworld.wolfram.com/QuadraticCurve.html">general bivariate quadratic curve</a> can be written in the implict form:

<div> $$ ax^2+2bxy+cy^2+2dx+2fy+g=0 $$ </div>

Quadratics represent conic sections: lines, ellipses, hyperbolas, and parabolas.  For example, with
the following coefficients:

<div> $$ a=1,\; b=0,\; c=1,\; d=0,\; f=0,\; g=-1 $$ </div>

Is the equation for a circle:

<div> $$ x^2+y^2=1 $$ </div>

Quadratic Bézier Curves
=======================

Bézier curves can't exactly express a circle, although a general
quadratic curve can.  What subset of conic sections are they?  The
quadratic Bézier is usually written as the set of parametric equations:

<div> $$ x(t) = (1-t)^2x_0+2t(1-t)x_1+t^2x_2 $$ </div>
<div> $$ y(t) = (1-t)^2y_0+2t(1-t)y_1+t^2y_2 $$ </div>

Deparameterizing the above (using Mathematica to eliminate t) leads to the
implicit form:

![Implicit Form of Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-implicit.svg)

Pulling out the coefficients to match the general form given above:

![Coefficients of Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-coefficients.svg)

The type of quadratic curve can be classified with the <a href="http://en.wikipedia.org/wiki/Conic_section#Discriminant_classification">discriminant</a>.

![Discriminant of Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-discriminant.svg)

The discriminant is always zero, meaning it is a parabola in the non-degenerate case.
I assume it should also be possible to show that degenerate case is coincident lines.
Therefore quadratic Béziers can only represent parabolas (and lines), but not 
ellipses or hyperbolas.  See also
<a href="http://en.wikipedia.org/wiki/Degenerate_conic">degenerate conic</a>.

Rational Quadratic Bézier Curves
================================

The Bézier equations <a href="http://en.wikipedia.org/wiki/B%C3%A9zier_curve#Rational_B.C3.A9zier_curves">can be extended</a> with a denominator of an additional Bernstein polynomial.
A weight parameter adds an additional level of control over the curvature.  When all of the
weights are equal it is equivalent to a non-rational Bézier curve.  Usually the endpoint weights are
normalized to one, because what matters is relative ratio of the weights, and this
normalization simplifies the curve to having a single weight w.

Redefining the Bernstein polynomials in terms of linear interpolation.

![Lerp definition of Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-lerp.svg)

An example evaluation of the rational equations with a weight of 0.5 and extended t
parameter range clearly shows an ellipse:

![w=0.5 Rational Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-w-ellipse.svg)

This form of the equations also clearly shows the concept of the rational Béziers, which is
similar in idea to homogeneous coordinates and the homogeneous divide.  The weight denominator is
interpolated with the same Bernstein polynomial as the x and y coordinates.

An example evaluation of the rational equations with a weight of 1.5 and extended t
parameter range clearly shows a hyperbola:

![w=1.5 Rational Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-w-hyperbola.svg)

The sign of the descriminant of the Rational Bézier depends only on the weight, because the
rest of the expression is squared and thus always positive.  Looking at a graph of the weight
contribution, the curve will be undefined at w of zero, a parabola at w of ±1, an ellipse between -1 and 1, and a hyperbola elsewhere.  See also the <a href="http://en.wikipedia.org/wiki/Eccentricity_%28mathematics%29">eccentricity</a> of a conic.

![Weight Factor in Discriminant of Rational Quadratic Bézier]({{ site.url }}/images/QuadraticCurves-bezier-w-graph.svg)

Some references that cover the classification of Rational Bézier Curves more formally are <a href="http://lgdv.cs.fau.de/get/477">here</a> and <a href="http://www.cs.mtu.edu/~shene/COURSES/cs3621/NOTES/spline/NURBS/RB-conics.html">here</a>.
