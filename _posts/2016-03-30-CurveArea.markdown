---
layout: post
title:  "Calculating Area of Closed Curves in ℝ²"
date:   2016-03-30 14:00:00
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


Introduction
============

There exists a lot of material covering arclength of Bézier curves, since it is a
common requirement in stroke dashing, animation, etc.  The arclength integral
doesn't have a closed form solution, so there has been a lot of work devoted to
different ways of computations and approximations.

There exists much less information, however, about the calculation of the area
of a Bézier curve.  At first it might seem like it would be more complicated
than the arclength, but as we'll see it actually has a very simple closed form
solution.


Green's Theorem
===============

The area of a closed
<a href="http://mathworld.wolfram.com/SimplyConnected.html">simply connected</a>
<a href="http://planetmath.org/piecewisesmooth">piecewise-smooth curve</a>
<a href="https://en.wikipedia.org/wiki/Green%27s_theorem#Area_calculation">
can be computed via Green's Theorem</a>.  I won't spend very much time covering
the maths, it is covered well in most Calculus books (ie
<a href="http://www.amazon.com/Calculus-James-Stewart/dp/1285740629">Stewart's Calculus</a>).
Green's Theorem shows that certain types of double integrals over a
2D region have an equivalent line integral.  In the case of computing the signed
area of a curve, you would simply integrate 1 over the region:

<div> $$ \iint_R 1 \,dx\,dy $$ </div>

Green's Theorem allows to this be stated equivalently as a few different line
integrals:

<div> $$ \oint_C x \,dy = -\oint_C y \,dx = \tfrac 12 \oint_C (x \,dy - y \,dx) $$ </div>

Note that this formulation is for the standard <a href="https://en.wikipedia.org/wiki/Curve_orientation">curve orientation</a> &mdash; the signed area is positive for counterclockwise
curves and negative for clockwise curves.  Some other derivations are in the
opposite order (ie <a href="https://tug.org/TUGboat/tb33-1/tb103jackowski.pdf">Jackowski</a>).

We can choose whichever form of the line integral is most convenient, they
should all simplify to the same result (more on that later).  Taking the first
form and writing it in terms of a typical parametric of t from 0 to 1:

<div> $$ \int_{0}^{1} y(t) \, x'(t) \,dt $$ </div>

For closed piecewise curves (closed paths with multiple segments), the area is
simply the sum of the area of the segments (keeping consistent orientation of
course).


Cubic Bézier Curves
===================

Here we run through the above 3 line integrals for cubic Bézier curves.

![Area of Cubic Bézier Segment]({{ site.url }}/images/CurveArea-bez-cubic-area-1.svg)

Note that while the three results are similar, they are not exactly the same:

![Differences in Forms of Cubic Bézier Segment Area]({{ site.url }}/images/CurveArea-bez-cubic-area-2.svg)

You can see that the different forms differ by some factor of x3y3 and x0y0.
However, since we have a closed curve, as we go along, the next segment's x0
and y0 will be the previous segment's x3 and y3.  So instead of adding the x0y0
and x3y3, only to then subtract it again on the next segment, the 3rd form
simplifies the expression by canceling the redundant terms across the segments,
and thus has 2 less components in the expression.  This is the same cancellation
that is common in the calculation of <a href="http://mathworld.wolfram.com/PolygonArea.html">
signed polygon area</a>, and we'll see this later.  As we extend this to process
paths with segments of mixed types of curves, it will be important to use the
same integral form across all types. The 3rd form we'll use matches the derivation by <a href="http://objectmix.com/graphics/133553-area-closed-bezier-curve.html">Kalle Rutanen</a>.

Below is a JavaScript function to compute the area of a cubic Bézier segment.

    function bez3_sauc(x0, y0, x1, y1, x2, y2, x3, y3) {
      return (  x3*(  -y0 - 3*y1 - 6*y2) -
              3*x2*(   y0 +   y1 - 2*y3) +
              3*x1*(-2*y0 +   y2 +   y3) +
                x0*( 6*y1 + 3*y2 +   y3)) / 20;
    }

Note that this calculation uses an expression arranged around the x components,
but there are perhaps cases where it is better numerically to arrange the
expression in another form.

Quadric Bézier Curves
=====================

The same process for quadratic Bézier curves.

![Area of Quadratic Bézier Segment]({{ site.url }}/images/CurveArea-bez-quad-area-1.svg)

    function bez2_sauc(x0, y0, x1, y1, x2, y2) {
      return (  x2*( -y0 - 2*y1) +
              2*x1*(  y2 -   y0) +
                x0*(2*y1 +   y2)) / 6;
    }

Again this calculation is arranged around the x terms.

Lines
=====

The same process for lines.

![Area of Line Segment]({{ site.url }}/images/CurveArea-bez-line-area-1.svg)

    function line_sauc(x0, y0, x1, y1) {
      return (x0*y1 - x1*y0) / 2;
    }

We have of course arrived at the 
<a href="https://en.wikipedia.org/wiki/Shoelace_formula#Definition">shoelace formula</a>,
specifically the form:

<div> $$ \mathbf{A} = {1 \over 2} \Big | \sum_{i=1}^{n} x_iy_{i+1}-x_{i+1}y_i \Big | $$ </div>
