---
layout: post
title:  "Calculating Area of Closed Curves in ℝ²"
date:   2016-03-30 14:00:00
categories: graphics
---

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/mathjax@3.2.2/es5/tex-mml-chtml.min.js"></script>
<script>
MathJax.Hub.Config({
    jax: ["input/TeX","output/HTML-CSS"],
    displayAlign: "left",
    displayIndent: "2em",
});
</script>

* TOC
{:toc}

Introduction
============

There exists a lot of material covering the arc length of Bézier curves, since
it is a common requirement in stroke dashing, animation, etc.  The arc length
integral for a cubic Bézier, for example, doesn't have a closed form solution,
so there has been many pieces of writing devoted to different computations and
approximations.

There exists much less information, however, about the calculation of the area
of a Bézier curve.  At first it might seem like it would be more complicated
than the arclength, but as we'll see it actually has a very simple closed form
solution.

There has been previously published derivations for the area of cubic Bézier
curves, and quadratic Bézier curves and lines are of course a subset of those
curves.

I have not found a derivation of the area of rational quadratic Bézier curves
(conic sections) however, and this article additionally attempts to derive a
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

<div> $$ \int_{0}^{1} x(t) \, y'(t) \,dt $$ </div>

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


Rational Quadratic Bézier Curves
================================

Rational quadratic Bézier curves, also sometimes referred to as conic sections,
add an additional weight parameter to expand the representable curves to
include [ellipses and hyperbolas]({% post_url 2015-01-15-QuadraticCurves %}).

Moving from a polynomial to a rational function makes the integral much more
involved.

![Area of Rational Quadratic Bézier Segment]({{ site.url }}/images/CurveArea-bez-conic-1.svg)

The result of the integral has some conditions attached, the graph above shows
that the result isn't valid when $$ w < -1 $$.  There are some singularities
at $$ \pm 1 $$, this corresponds to when the curve [is a parabola]({% post_url 2015-01-15-QuadraticCurves %}).  Anyway when $$ w = 1 $$ the curve is equivalent to a non-rational
quadratic Bézier, so it is a case we already have a solution for.

We can break the problem into three ranges:

![Valid ranges of w]({{ site.url }}/images/CurveArea-bez-conic-3.svg)

Although it would be nice to support negative weights, most graphics systems
limit weights to being positive.  As stated by Ron Goldman: "Negative weights,
however, may introduce singularities even if we restrict the parameter domain,
so negative weights are generally avoided."

Additionally there are some problems directly implementing the above equation,
as some of the subexpressions are complex.  We can manually rearrange the
expression so the subexpressions are all real on $$ 0 < w < 1 $$:

![Conic equation rearranged for 0<w<1]({{ site.url }}/images/CurveArea-bez-conic-2.svg)

We still have another difficulty in terms of implementation.  As w approaches 1,
the denominator will go towards zero and thus the fraction will approach infinity.
However we know the answer should be approaching that of a non-rational
quadratic Bézier as we approach 1.  In order to implement this stably we need
to better understand the asymptotic behaviour.  The key here is to understand
the <a href="https://en.wikipedia.org/wiki/Inverse_trigonometric_functions#Infinite_series">
series expansion of arctan</a>.

![Conic equation arctan behaviour]({{ site.url }}/images/CurveArea-bez-conic-5.svg)

As w approaches 1 the argument of arctan approaches 0, and as x approaches 0
arctan(x) approaches x.  We can use an approximation based on a truncated
series expansion to cancel out the denominator and produce a stable rational
expression for use when w is near 1.

![Conic equation using arctan expansion]({{ site.url }}/images/CurveArea-bez-conic-6.svg)

Similarly we can rewrite the equation in terms of the inverse hyperbolic tangent
so that all subexpressions will be real for $$ w > 1 $$.

![Conic equation rearranged for w>1]({{ site.url }}/images/CurveArea-bez-conic-4.svg)

<!--
<div> $$
\frac{1}{2 \sqrt{1-w^2} \left(1-w^2\right)} \left(2 \tan ^{-1}\left(\frac{\sqrt{w-1}}{\sqrt{-w-1}}\right) (-w \text{x0} \text{y2}+w \text{x2} \text{y0}+\text{x0} \text{y1}+\text{x1} (\text{y2}-\text{y0})-\text{x2} \text{y1})-\sqrt{1-w^2} (w (\text{x0} \text{y1}-\text{x1} \text{y0}+\text{x1} \text{y2}-\text{x2} \text{y1})-\text{x0} \text{y2}+\text{x2} \text{y0})\right)
 $$ </div>
 -->
