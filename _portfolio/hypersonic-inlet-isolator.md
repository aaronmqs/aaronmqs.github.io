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

My colleagues at Notre Dame tested an axisymmetric inlet/isolator model at Mach 5.7 and measured
the pressure on the inner wall of the isolator with fast pressure-sensitive paint [1]. Paint gives
the pressure on the wall only. After the experiments, **I computed the steady flow inside the
model** to give the off-wall context of the measurements: the shocks, expansions, and separated
regions inside the duct, and the flow conditions at the isolator entrance.

## The computational model

### Geometry and domain

The model is a conical inlet with a 9° half-angle, followed by a straight isolator. The capture
diameter is 63.5 mm and the isolator diameter is 50.8 mm. The free stream is at Mach 5.7, with a
total temperature of 429.96 K, a static pressure of 506.48 Pa, and a unit Reynolds number of
7.1 × 10⁶ /m.

![The inlet/isolator on a plane through the axis](/images/isolator/isolator_overview.svg)

*The inlet and isolator at true scale, cut through the axis. The flow is axisymmetric, so only
the upper half is computed; the lower half is its mirror image.*
{: .caption}

The coordinate $$x$$ is the one used in the paper: it starts at the **isolator entrance**, 64.7 mm
downstream of the capture plane. The isolator entrance is not the shoulder, where the conical
wall ends: the shoulder is about 25 mm upstream of it.

![Computational domain and boundary conditions](/images/isolator/isolator_domain.svg)

*The computational domain and its boundary conditions, on a plane through the axis. The
averaging plane $$A$$ at the isolator entrance is used for the approach conditions below.*
{: .caption}

### Numerical method

- **Equations:** axisymmetric (no swirl) compressible Navier–Stokes, ideal gas, viscosity from
  Sutherland's law.
- **Discretization:** interior-penalty discontinuous Galerkin (DG) method, polynomial degree
  $$p = 4$$ on 151,552 triangles, which gives 11,366,400 degrees of freedom.
- **Shocks:** a PDE-based artificial viscosity, driven by a dilation-based shock sensor, keeps the
  high-order solution stable at the shocks.
- **Solver:** pseudo-transient continuation with an adaptive time step.
- **Code and hardware:** Talon, our lab's in-house high-order CFD code, written in Julia. The
  final solve ran on 64 MPI ranks at the
  [Notre Dame Center for Research Computing (CRC)](https://crc.nd.edu/).

## Results

The contours below show the flow on a plane through the axis. The figures are long and thin, like
the isolator: **scroll sideways to see the whole length.**

<div class="hscroll-fig">
  <div class="hscroll">
    <img src="/images/isolator/isolator_mach.png" alt="Mach number contour in the inlet and isolator">
  </div>
  <img class="cbar" src="/images/isolator/isolator_mach_cbar.png" alt="Mach number color scale, 0 to 6">
</div>

*Mach number. The conical shock from the inlet leading edge reflects from the axis and from the
wall along the whole isolator. Adapted from [1], © 2026 American Institute of Aeronautics and Astronautics.*
{: .caption}

<div class="hscroll-fig">
  <div class="hscroll">
    <img src="/images/isolator/isolator_pressure.png" alt="Static pressure contour in the inlet and isolator">
  </div>
  <img class="cbar" src="/images/isolator/isolator_pressure_cbar.png" alt="Pressure color scale, 1 to 40 times the free-stream pressure">
</div>

*Static pressure divided by the free-stream pressure (log scale). Adapted from [1], © 2026 American Institute of Aeronautics and Astronautics.*
{: .caption}

The main flow features are:

- The **conical shock** from the inlet leading edge reflects through the isolator.
- **Expansion waves** start at the inlet shoulder.
- Two **separation bubbles** form on the wall, at about $$x = 0$$ to 80 mm and $$x = 200$$ to
  250 mm. Each one generates a separation shock, which also reflects downstream.

The paper uses the first separation bubble to explain a measurement: once the shock train moves
upstream of about $$x = 120$$ mm, it speeds up, because the separated flow cannot sustain a large
pressure rise across it.

Because of the shocks and expansions, the flow at the isolator entrance is far from uniform. So
the approach conditions are given as **mass-weighted averages** over the entrance plane $$A$$:

$$\bar{M} = \frac{\int_A M\,\rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}{\int_A \rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}, \qquad \bar{p} = \frac{\int_A p\,\rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}{\int_A \rho\,|\mathbf{v}\cdot\mathrm{d}\mathbf{A}|}.$$

The result is $$\bar{M} = 4.63$$ and $$\bar{p} = 2.98\,p_\infty$$. The paper uses these values as
the approach conditions of the isolator, for example to compare the measured pressure rise across
the shock train with the pressure rise across a normal shock at the isolator entrance.

## Mesh and degree convergence

The solution was computed on four nested meshes (each one splits every triangle of the previous
mesh into four) with polynomial degrees from 2 to 5. The mesh was refined ($$h$$) and the degree
was increased ($$p$$) until the change in both averages was below 1%. Between the last two levels,
$$\bar{M}$$ changes by 0.27% and $$\bar{p}$$ by 0.81%.

![hp convergence](/images/isolator/isolator_hp_convergence.png)

*Mass-weighted Mach number (left) and pressure (right) at the isolator entrance, against the
number of degrees of freedom. The marker shape and color give the mesh, the label gives the
polynomial degree. The count includes the five fields of the solution (four flow variables and
the artificial viscosity). Data from [1], © 2026 American Institute of Aeronautics and Astronautics.*
{: .caption}

## Reference

1. A. N. Bustard, B. L. Bemis, A. Marques, M. J. Zahr, and T. J. Juliano, *Global time-resolved
   measurements of inlet/isolator unstart induced by mass injection*, Journal of Propulsion and
   Power (2026), [doi:10.2514/1.B40160](https://arc.aiaa.org/doi/10.2514/1.B40160). The
   simulations on this page are in Section IV.
