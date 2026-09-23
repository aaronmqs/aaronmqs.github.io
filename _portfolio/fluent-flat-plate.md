---
title: "Laminar flat plate in Ansys Fluent: verification against Blasius"
excerpt: "<em>Work in progress.</em> Velocity profiles at five stations collapse onto the exact Blasius solution within 0.29%. Skin friction and wall heat flux are verified the same way on three meshes.<br/><img src='/images/flatplate/flatplate_similarity.png' width='400'>"
collection: portfolio
---

**Work in progress.** This page is not final. The text and the figures can change.
{: .notice--warning}

I set up, solved, and verified a steady laminar flat-plate boundary layer with heat transfer in
**Ansys Fluent**.

<div class="schematic-pair">
  <img src="/images/flatplate/flatplate_side.svg" alt="Side view of the flat plate: uniform flow from the left, boundary layer growing along the heated plate">
  <img src="/images/flatplate/flatplate_3d.svg" alt="Three-dimensional view of the flat plate with flow passing over its heated top surface">
</div>
*Side and three-dimensional views. The plate is drawn with thickness for visibility only.*

Steady laminar flow over an isothermal flat plate of zero thickness, at
$$\mathrm{Re}_L = 10^5$$, with heat transfer. The wall is at 350 K in a 300 K stream. The fluid is
air-like: $$\mathrm{Pr} = 0.72$$, with constant density and viscosity, so the Blasius and
Pohlhausen similarity solution is exact and not approximate.

The quantities of interest are the skin friction $$C_f$$, the wall heat flux $$q_w$$, and the
velocity profile across the layer. The first two integrate to the plate drag and the mean cooling
rate.

Each is measured against the similarity solution on three systematically refined meshes, which
separates the discretization error from the terms that boundary-layer theory omits.

## The exact solution

For a laminar layer with zero pressure gradient and constant fluid properties, the velocity
profiles at all stations have the same shape when the wall distance is scaled by the local layer
thickness. With the similarity variable

$$\eta = y\sqrt{\frac{U_\infty}{\nu x}}, \qquad \frac{u}{U_\infty} = f'(\eta), \qquad \theta(\eta) = \frac{T - T_w}{T_\infty - T_w},$$

the flow equations reduce to two ordinary differential equations, due to Blasius (velocity) and
Pohlhausen (temperature):

$$f''' + \tfrac{1}{2} f f'' = 0, \qquad f(0) = f'(0) = 0, \quad f'(\infty) = 1,$$

$$\theta'' + \tfrac{1}{2}\,\mathrm{Pr}\, f\, \theta' = 0, \qquad \theta(0) = 0, \quad \theta(\infty) = 1.$$

I integrated both numerically. The wall values give skin friction and heat transfer along the
plate, with $$\mathrm{Re}_x = U_\infty x/\nu$$:

$$C_f = \frac{\tau_w}{\tfrac{1}{2}\rho U_\infty^2} = \frac{2 f''(0)}{\sqrt{\mathrm{Re}_x}} = \frac{0.664115}{\sqrt{\mathrm{Re}_x}}, \qquad \mathrm{Nu}_x = \frac{q_w\, x}{k\,(T_w - T_\infty)} = \theta'(0)\sqrt{\mathrm{Re}_x} = 0.295635\sqrt{\mathrm{Re}_x}.$$

The heat-transfer constant is for $$\mathrm{Pr} = 0.72$$. The common textbook correlation
$$0.332\,\mathrm{Pr}^{1/3}$$ gives 0.297565, which is 0.65% off. That is larger than the
discretization error measured below, so a correlation cannot serve as the reference here.

## Setup in Fluent

| Item | Value |
|---|---|
| Software | Ansys Fluent 2025 R2, 2D, steady, laminar, energy equation on |
| Flow | $$U_\infty = 1$$ m/s, $$\mathrm{Re}_L = 10^5$$, plate length $$L = 1$$ m |
| Fluid | air-like, constant properties: $$\rho = 1$$ kg/m³, $$\mu = 10^{-5}$$ kg/(m·s), $$\mathrm{Pr} = 0.72$$ |
| Temperatures | stream 300 K, isothermal wall 350 K |
| Domain | $$x \in [-0.5, 1.0]$$ m, $$y \in [0, 0.2]$$ m |
| Solver | pressure-based, coupled; second-order upwind for momentum and energy |

Real air at 300 K has $$\rho = 1.18$$ kg/m³ and $$\mu = 1.85 \times 10^{-5}$$ kg/(m·s), and both
change with temperature. Here they are rounded and held constant. That keeps the problem exactly
the one the similarity solution describes, so the reference stays exact and any difference is the
solver's. The values also make $$\mathrm{Re}_x = x/10^{-5}$$, with $$x$$ in meters. The Prandtl
number is that of air, because it sets the heat transfer.

![Domain and boundary conditions](/images/flatplate/flatplate_domain.svg)
*The domain and its boundary conditions. The inset shows what "graded cells" means: the cells are
smallest at the wall and at the leading edge, where the solution changes fastest.*

Boundary conditions:

- **Inlet** ($$x = -0.5$$): uniform velocity and temperature.
- **Symmetry** on $$y = 0$$ ahead of the plate ($$x < 0$$), so the stream reaches the leading
  edge undisturbed.
- **Wall** on $$y = 0$$, $$0 \le x \le 1$$: no slip, fixed temperature.
- **Pressure outlets** at the exit ($$x = 1$$) and on the top ($$y = 0.2$$).

**The top boundary matters.** I first set the top to symmetry, which looked harmless: the domain
is twelve boundary-layer thicknesses tall. But a symmetry plane is a slip wall. The fluid that the
boundary layer pushes upward cannot leave, so the stream above the plate accelerates, and the wall
shear comes out too high. The error grew along the plate, from $$+2.8\%$$ to $$+10.9\%$$, which
looks like a mesh problem. Dividing each error by the local displacement thickness over the
channel height gave a constant, 3.3 to 4.0, which is the signature of blockage. A pressure outlet
on the top removed it.

**The reverse-flow problem.** The boundary layer displaces fluid upward, so fluid leaves through
the top boundary. With default settings, Fluent let fluid *enter* through 30–51% of the top
outlet at every iteration. The continuity residual stalled at $$1.4 \times 10^{-1}$$ for 1000
iterations. The only diagnosis was a console note, "Reversed flow on 83 faces", printed as
information, not as a warning. The fix was one option, **Prevent Reverse Flow** on the outlet's
Momentum tab, which is off by default. With it on, continuity dropped to $$8 \times 10^{-12}$$ in
400 iterations.

## Mesh

The mesh is structured quadrilaterals in three levels. Each level has twice the cells of the
previous one in each direction:

| Level | Cells | First cell height at the wall |
|---|---|---|
| Coarse | $$(60 + 30) \times 30 = 2{,}700$$ | $$3.9 \times 10^{-4}$$ m |
| Medium | $$(120 + 60) \times 60 = 10{,}800$$ | $$2.0 \times 10^{-4}$$ m |
| Fine | $$(240 + 120) \times 120 = 43{,}200$$ | $$1.0 \times 10^{-4}$$ m |

The cells are clustered toward the wall, where the velocity changes fastest, and toward the leading
edge, where the layer starts. The grading factors are the same on all levels, so the three meshes
differ only in cell size. This lets the error be measured as the mesh is refined.

## Solution

![Streamwise velocity](/images/flatplate/flatplate_u.png)
*Streamwise velocity $$u$$, full domain to scale.*

![Gauge pressure](/images/flatplate/flatplate_p.png)
*Gauge pressure. It is flat everywhere except at a peak at the leading edge. A flat pressure is
the assumption that the Blasius solution rests on.*

![Boundary layer close-up](/images/flatplate/flatplate_u_bl.png)
*The same velocity field near the wall, vertical scale $$\times 4$$. The layer grows from zero
thickness at $$x = 0$$, as $$\sqrt{x}$$.*

## Results

The error in any computed quantity $$\phi$$ is $$E = (\phi - \phi_\mathrm{exact})/\phi_\mathrm{exact}$$,
in percent. Pass criteria are judged for $$x \ge 0.1$$ m. Closer to the leading edge, the exact
solution itself is singular, and no mesh resolves it.

### Skin friction

Skin friction gives the drag on the plate. It depends on the velocity gradient at the wall, so
it is the most sensitive test of the near-wall mesh.

![Skin friction](/images/flatplate/flatplate_cf.png)
*Top: $$C_f$$ on the fine mesh against Blasius. Middle and bottom: error in percent, with the
$$\pm 2\%$$ band shaded. The large error near $$x = 0$$ comes from the $$x^{-1/2}$$ singularity of
the exact solution at the leading edge, not from a mesh failure.*

### Wall heat flux

The wall heat flux is the cooling rate, the number an engineer designs with. It depends on the
temperature gradient at the wall, and it tests the energy equation.

![Wall heat flux](/images/flatplate/flatplate_qw.png)
*Wall heat flux on the fine mesh against the exact $$65.339/\sqrt{x}$$ W/m², with the $$\pm 3\%$$
band shaded. The error is about $$-0.2\%$$ and flat over the judged range.*

### Velocity profiles

Wall values test only the first cells. The profiles test the whole layer.

![Velocity profiles](/images/flatplate/flatplate_similarity.png)
*Velocity profiles at five stations, medium mesh, plotted against $$\eta$$. They collapse onto
the Blasius profile within 0.29%.*

### Mesh convergence

A match with the exact solution on one mesh can be luck. Three meshes show how the error shrinks
with the cell size, and whether it shrinks at the rate the numerical scheme promises.

![Mesh convergence](/images/flatplate/flatplate_convergence.png)
*Error in $$C_f$$ and $$q_w$$ on all three meshes. The dotted line is the error extrapolated to
zero cell size.*

The observed order of accuracy is 1.56 to 2.01, close to the formal second order of the scheme.
The grid convergence index (GCI), an estimate of the remaining numerical error, is at most 0.077%.
The extrapolated errors, $$-0.45\%$$ for $$C_f$$ and $$-0.17\%$$ for $$q_w$$, are much larger than
the GCI. So the remaining difference is not numerical error. It is physics that the Navier–Stokes
solution contains and boundary-layer theory leaves out: terms of order $$\mathrm{Re}_x^{-1/2}$$.

### Summary

| Quantity | Criterion | Fine mesh | Result |
|---|---|---|---|
| Skin friction error, $$x \geq 0.1$$ | $$\lvert E \rvert < 2\%$$ | $$+0.20\%$$ to $$-1.15\%$$ | pass |
| Wall heat flux error, $$x \geq 0.1$$ | $$\lvert E \rvert < 3\%$$ | $$-0.13\%$$ to $$-0.36\%$$ | pass |
| Velocity profile error, 5 stations | $$< 1\%$$ | $$0.29\%$$ | pass |
| Plate drag coefficient | — | $$4.2136 \times 10^{-3}$$ ($$+0.32\%$$) | |
| Mean wall heat flux | — | 130.68 W/m² ($$+0.002\%$$) | |

## Issues observed

This was my first case in Fluent. Several problems produced no error message, and I caught each
one only by predicting a number and checking it.

- **"Converged" did not mean converged.** With the default residual criteria, Fluent reported
  convergence while the local skin friction at the trailing edge was still 17.9% off. A monitor on
  the drag coefficient caught it: the value still moved in the third significant figure.
- **An unconverged solution imitated physics.** Before convergence, the edge velocity rose along
  the plate by an amount that matched a real blockage effect within half a percent. It was an
  artifact.
- **Mesh settings were silently overridden.** The option *Blend to Neighbors* is on by default for
  every edge sizing. It changed a requested 140 columns to 153, with no warning. I found it by
  factoring the cell count in the mesh statistics.
- **A wall-spacing error was invisible in the interface.** A grading factor of 20 where 70 was
  needed made the first wall cell 2.6 times too large. I found it from the exported node
  coordinates.
- **A passing average hid a wrong boundary condition.** With the symmetry top, the mean wall heat
  flux was still only 1.04% off, because the average is weighted toward the leading edge, where
  the blockage is small. The local profile exposed the error at once.
- **Eleven silent exits.** Fluent closed with no message eleven times on this small 2D case, so
  each restart had to be checked against a known number.

### Possible improvements

- Rerun with real air properties at the film temperature (325 K) instead of the air-like values.
- Extract the velocity profiles on the fine mesh. A mesh replacement silently dropped the sampling
  lines, so the profiles are one level behind the other results.
- Next cases: variable properties and compressibility, then a turbulent flat plate.
