---
title: "Mach 5.7 axisymmetric inlet/isolator: high-order CFD"
excerpt: "Steady viscous simulation of a hypersonic inlet/isolator with a high-order discontinuous Galerkin solver on 11.4 million degrees of freedom, run in parallel with MPI. Published in the Journal of Propulsion and Power (2026).<br/><img src='/images/isolator/isolator_thumb.png' width='400'>"
collection: portfolio
---

{% include toc title="Contents" %}

## The problem

An inlet/isolator is the front part of a dual-mode scramjet engine. The inlet captures the air and
compresses it. The isolator is a straight duct that separates the inlet from the combustor. If the
pressure in the combustor becomes too high, a shock train moves upstream through the isolator and
the inlet *unstarts*: it no longer captures the flow.

My colleagues at Notre Dame measured the pressure on the inner wall of an axisymmetric
inlet/isolator model at Mach 5.7, with fast pressure-sensitive paint [1]. Paint gives the pressure
on the wall only. **I computed the steady flow inside the model**, to show the flow structures away
from the wall and to give the flow conditions at the isolator entrance.

The model is a conical inlet with a 9° half-angle, followed by a straight isolator. The capture
diameter is 63.5 mm and the isolator diameter is 50.8 mm. The free stream is at Mach 5.7, with a
total temperature of 429.96 K, a static pressure of 506.48 Pa, and a unit Reynolds number of
7.1 × 10⁶ /m.

## The computational model

- **Equations:** axisymmetric (no swirl) compressible Navier–Stokes, ideal gas, viscosity from
  Sutherland's law.
- **Domain:** the interior of the inlet and isolator, plus a short region upstream of the inlet.
- **Boundary conditions:** supersonic inflow and outflow; no-slip isothermal walls; symmetry on
  the axis ($$r = 0$$).
- **Discretization:** interior-penalty discontinuous Galerkin (DG) method, polynomial degree
  $$p = 4$$ on 151,552 triangles, which gives 11,366,400 degrees of freedom.
- **Shocks:** a PDE-based artificial viscosity, driven by a dilation-based shock sensor, keeps the
  high-order solution stable at the shocks.
- **Solver:** pseudo-transient continuation with an adaptive time step, in Talon, our in-house
  Julia code. The final solve ran on 64 MPI ranks on the Notre Dame cluster.

## Results

The contours below show the flow on a plane through the axis. The coordinate $$x$$ starts at the
isolator entrance. The figures are long and thin, like the isolator: **scroll sideways to see
the whole length.**

<div class="hscroll-fig">
  <div class="hscroll">
    <img src="/images/isolator/isolator_mach.png" alt="Mach number contour in the inlet and isolator">
  </div>
  <img class="cbar" src="/images/isolator/isolator_mach_cbar.png" alt="Mach number color scale, 0 to 6">
</div>

*Mach number. The conical shock from the inlet leading edge reflects from the axis and from the
wall along the whole isolator.*
{: .caption}

<div class="hscroll-fig">
  <div class="hscroll">
    <img src="/images/isolator/isolator_pressure.png" alt="Static pressure contour in the inlet and isolator">
  </div>
  <img class="cbar" src="/images/isolator/isolator_pressure_cbar.png" alt="Pressure color scale, 1 to 40 times the free-stream pressure">
</div>

*Static pressure divided by the free-stream pressure (log scale).*
{: .caption}

The main flow features are:

- The **conical shock** from the inlet leading edge reflects through the isolator.
- **Expansion waves** start at the inlet shoulder, where the cone meets the straight duct.
- Two **separation bubbles** form on the wall, at about $$x = 0$$ to 80 mm and $$x = 200$$ to
  250 mm. Each one generates a separation shock, which also reflects downstream.

Because of these shocks and expansions, the flow at the isolator entrance is far from uniform. So
the entrance conditions are given as **mass-weighted averages** over the entrance plane $$A$$:

$$\bar{M} = \frac{\int_A M\,\rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}{\int_A \rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}, \qquad \bar{p} = \frac{\int_A p\,\rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}{\int_A \rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}.$$

The result is $$\bar{M} = 4.64$$ and $$\bar{p} = 2.98\,p_\infty$$. The experiment uses these
values as the approach conditions of the isolator.

## Mesh and degree convergence

The mesh was refined and the polynomial degree was increased ($$hp$$ refinement) until the change
in both averages was below 1%. Between the last two levels, $$\bar{M}$$ changes by 0.27% and
$$\bar{p}$$ by 0.81%.

![hp convergence](/images/isolator/isolator_hp_convergence.png)

*Mass-weighted Mach number (left) and pressure (right) at the isolator entrance, against the
number of degrees of freedom. The red marker is the final solution.*
{: .caption}

## Reference

1. A. N. Bustard, B. L. Bemis, A. Marques, M. J. Zahr, and T. J. Juliano, *Global time-resolved
   measurements of inlet/isolator unstart induced by mass injection*, Journal of Propulsion and
   Power (2026), [doi:10.2514/1.B40160](https://arc.aiaa.org/doi/10.2514/1.B40160). The
   simulations on this page are in Section IV.
