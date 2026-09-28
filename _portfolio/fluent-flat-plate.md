---
title: "Laminar flat plate in Ansys Fluent: verification against Blasius"
excerpt: "<em>Work in progress.</em> Velocity profiles at five stations collapse onto the exact Blasius solution within 0.32%. Skin friction and wall heat flux are verified the same way on three meshes.<br/><img src='/images/flatplate/flatplate_similarity.png' width='400'>"
collection: portfolio
---

**Work in progress.** This page is not final. The text and the figures can change.
{: .notice--warning}

{% include toc title="Contents" %}

## The problem

I set up, solved, and verified a steady laminar flat-plate boundary layer with heat transfer in
**Ansys Fluent**.

The case is a steady laminar flow over an isothermal flat plate (Figure 1),
at $$\mathrm{Re}_L = 10^5$$. The wall is at 350 K in a 300 K stream. The fluid is
air-like: $$\mathrm{Pr} = 0.72$$, with constant density and viscosity, so the Blasius and
Pohlhausen similarity solution is exact and not approximate.

<div class="schematic-pair">
  <img src="/images/flatplate/flatplate_side.svg" alt="Side view of the flat plate: uniform flow from the left, boundary layer growing along the heated plate">
  <img src="/images/flatplate/flatplate_3d.svg" alt="Three-dimensional view of the flat plate with flow passing over its heated top surface">
</div>

*Side and three-dimensional views. The plate is drawn with thickness for visibility only.*
{: .caption}

The quantities of interest are the skin friction $$C_f$$, the wall heat flux $$q_w$$, and the
velocity profile across the boundary layer. The first two integrate to the plate drag and the mean cooling
rate.

All three are measured against the similarity solution.

## The computational model

### Domain and boundary conditions

Figure 2 shows the computational domain and its boundary conditions.

![Domain and boundary conditions](/images/flatplate/flatplate_domain.svg)

*The domain and its boundary conditions. The inset shows what "graded cells" means: the cells are
smallest at the wall and at the leading edge, where the solution changes fastest.*
{: .caption}

Boundary conditions:

- **Inlet** ($$x = -0.5$$): uniform velocity and temperature.
- **Symmetry** on $$y = 0$$ ahead of the plate ($$x < 0$$), so the stream reaches the leading
  edge undisturbed.
- **Wall** on $$y = 0$$, $$0 \le x \le 1$$: no slip, fixed temperature.
- **Pressure outlets** at the exit ($$x = 1$$) and on the top ($$y = 0.2$$).

**Prevent Reverse Flow** (off by default) was turned on at the top outlet. The boundary layer
pushes fluid out through the top. Without the option, fluid also entered there, and the
continuity residual stalled at $$1.4 \times 10^{-1}$$. With it, the residual reached
$$8 \times 10^{-12}$$ in 400 iterations.

### Mesh

The mesh is structured quadrilaterals: 120 cells along the plate plus 60 ahead of it, times 60
normal to the wall, so $$(120 + 60) \times 60 = 10{,}800$$ cells. The first cell at the wall is
$$2.0 \times 10^{-4}$$ m high. Cells are clustered toward the wall and the leading edge.

### Setup in Fluent

| Item | Value |
|---|---|
| Software | Ansys Fluent 2025 R2, 2D, steady, laminar, energy equation on |
| Flow | $$U_\infty = 1$$ m/s, $$\mathrm{Re}_L = 10^5$$, plate length $$L = 1$$ m |
| Fluid | air-like, constant properties: $$\rho = 1$$ kg/m³, $$\mu = 10^{-5}$$ kg/(m·s), $$\mathrm{Pr} = 0.72$$ |
| Temperatures | stream 300 K, isothermal wall 350 K |
| Domain | $$x \in [-0.5, 1.0]$$ m, $$y \in [0, 0.2]$$ m |
| Solver | pressure-based, coupled; second-order upwind for momentum and energy |

Real air at 300 K has $$\rho = 1.18$$ kg/m³ and $$\mu = 1.85 \times 10^{-5}$$ kg/(m·s), and both
change with temperature. Here they are rounded and held constant, which is the assumption of the
similarity solution. The comparison then contains only the discretization error, the terms that
boundary-layer theory omits, and the effect of the finite domain. The values also make $$\mathrm{Re}_x = x/10^{-5}$$, with $$x$$ in meters. The Prandtl
number is kept at the value for air, so the heat-transfer result applies to air.

### Solution and quantities of interest

The error in any computed quantity $$\phi$$ is $$E = (\phi - \phi_\mathrm{exact})/\phi_\mathrm{exact}$$,
in percent. The error is checked only for $$x \ge 0.1$$ m, because the exact solution is singular
at the leading edge and no mesh resolves it.

In the figures below, each marker is a value that Fluent interpolates from the mesh nodes, and
each node value is an average of the surrounding cells. No marker is at a cell center.

#### Skin friction

The skin friction coefficient is the wall shear stress, made nondimensional with the dynamic
pressure of the stream:

$$C_f = \frac{\tau_w}{\tfrac{1}{2}\rho U_\infty^2}, \qquad \tau_w = \mu \left.\frac{\partial u}{\partial y}\right|_{y=0}.$$

The shear stress is the force per unit area that the fluid applies along the plate, so its
integral over the plate is the drag. The average of $$C_f$$ over the plate is the drag coefficient, $$C_d = \frac{1}{L}\int_0^L C_f\,\mathrm{d}x$$. Because
$$C_f$$ depends on the velocity gradient at the wall, its error shows if the cells next to the
wall are small enough.

The markers in the figure are the mesh nodes on the wall. Near the leading edge the error is large,
up to $$+50\%$$ at the first node. It comes from the $$x^{-1/2}$$ singularity of the exact
solution, not from a mesh failure.

![Skin friction](/images/flatplate/flatplate_cf.png)

*Top: $$C_f$$ against Blasius. Bottom: error in percent, with the $$\pm 2\%$$ band shaded. The
first two points ($$+50\%$$ and $$+17\%$$) are above the range of the bottom plot.*
{: .caption}

#### Wall heat flux

The wall heat flux is the heat per unit area that the plate gives to the fluid, by conduction
through the fluid at the wall:

$$q_w = -k \left.\frac{\partial T}{\partial y}\right|_{y=0}.$$

It is the cooling rate, the number an engineer designs with. Its integral over the plate is the
total heat transfer. Because $$q_w$$ depends on the temperature gradient at the wall, it tests the
energy equation. With the Pohlhausen solution, the exact value is

$$q_w = k\,(T_w - T_\infty)\,\theta'(0)\sqrt{\frac{U_\infty}{\nu x}} = \frac{65.339}{\sqrt{x}}\ \mathrm{W/m^2},$$

with $$k = 0.013978$$ W/(m·K), the value that gives $$\mathrm{Pr} = 0.72$$ with Fluent's
$$c_p = 1006.43$$ J/(kg·K).

As for $$C_f$$, the markers are the mesh nodes on the wall. Over the judged range, the error is
between $$-0.27\%$$ and $$-0.54\%$$.

![Wall heat flux](/images/flatplate/flatplate_qw.png)

*Top: $$q_w$$ against the exact solution. Bottom: error in percent, with the $$\pm 3\%$$ band
shaded. The first point ($$+48\%$$) is above the range of the bottom plot.*
{: .caption}

#### Velocity profiles

Wall values test only the first cells. The profiles test the whole layer.

The velocity is divided by the local edge velocity $$U_e$$, so the comparison tests the shape of
the profile. Each marker is a point where a vertical sampling line crosses a mesh line. Fluent
interpolates the value there from the two mesh nodes of the crossed edge. The largest error is
$$0.32\%$$.

![Velocity profiles](/images/flatplate/flatplate_similarity.png)

*Top: velocity profiles at five stations against the Blasius profile $$f'(\eta)$$. Bottom: error
$$u/U_e - f'(\eta)$$ in percent, with the $$\pm 1\%$$ band shaded.*
{: .caption}

#### Mesh convergence

A match with the exact solution on one mesh can be luck. To check convergence, the case was
also solved on a coarser and a finer mesh, with half and twice the cells in each direction and the
same grading:

| Mesh | Cells | First cell height at the wall |
|---|---|---|
| Coarser | $$(60 + 30) \times 30 = 2{,}700$$ | $$3.9 \times 10^{-4}$$ m |
| Results above | $$(120 + 60) \times 60 = 10{,}800$$ | $$2.0 \times 10^{-4}$$ m |
| Finer | $$(240 + 120) \times 120 = 43{,}200$$ | $$1.0 \times 10^{-4}$$ m |

**Two kinds of error.** The difference between Fluent and the exact solution has two parts. The
*mesh error* comes from the finite cell size, and it goes to zero as the cells shrink. The rest
does not depend on the mesh: the terms that boundary-layer theory omits, and the effect of the
finite domain. No mesh refinement removes it. The three meshes separate the two parts.

**The model.** For a scheme of order $$p$$, the error on a mesh with cell size $$h$$ behaves as

$$E(h) = E_0 + C\,h^p,$$

where $$E_0$$ is the error at zero cell size and $$C\,h^p$$ is the mesh error. Each mesh here has
half the cell size of the previous one, so each refinement divides the mesh error by $$2^p$$. With
the errors $$E_c$$, $$E_m$$, $$E_f$$ on the coarse, medium, and fine meshes, the three unknowns
follow (Richardson extrapolation [4, 5]):

$$2^p = \frac{E_c - E_m}{E_m - E_f}, \qquad D = \frac{E_m - E_f}{2^p - 1}, \qquad E_0 = E_f - D.$$

$$D$$ is the mesh error that remains on the fine mesh. The cell size itself cancels, so only the
ratio of 2 between the meshes enters. That matters here, because a graded mesh has no single cell
size. The grid convergence index, $$\mathrm{GCI} = 1.25\,\lvert D \rvert$$, is a conservative
estimate of this remaining mesh error.

**Example: the drag coefficient.** $$C_d$$ integrates the skin friction over the whole plate.

| Mesh | Coarser | Results above | Finer |
|---|---|---|---|
| Error in $$C_d$$ | $$-0.151\%$$ | $$+0.199\%$$ | $$+0.318\%$$ |

The changes between the meshes are $$+0.350$$ and $$+0.119$$, so $$2^p = 2.94$$ and $$p = 1.56$$.
The fine mesh still has a mesh error $$D = -0.061\%$$, so $$E_0 = 0.318 + 0.061 = +0.38\%$$, with
a GCI of 0.077%.

Two facts follow. First, the mesh error on the fine mesh is small. Second, the error does not go
to zero: with finer meshes, $$C_d$$ moves to $$+0.38\%$$, farther from Blasius. The middle mesh
looks the closest ($$+0.20\%$$) only because its negative mesh error cancels part of the positive
$$E_0$$. This is why the signs of the errors are kept: with absolute values, the finer mesh would
look worse.

![Convergence of the plate drag and the mean heat flux](/images/flatplate/flatplate_integrated.png)

*Error in $$C_d$$ and $$\bar{q}_w$$ against the cell size, relative to the finest mesh. The curve
$$E_0 + C h^p$$ passes through the three meshes. The circle is its value at zero cell size.*
{: .caption}

**All quantities.** The same procedure gives:

| Quantity | Coarser | Results above | Finer | $$p$$ | $$E_0$$ | GCI |
|---|---|---|---|---|---|---|
| Plate drag coefficient $$C_d$$ | $$-0.151\%$$ | $$+0.199\%$$ | $$+0.318\%$$ | 1.56 | $$+0.380\%$$ | 0.077% |
| Mean wall heat flux $$\bar{q}_w$$ | $$-0.335\%$$ | $$-0.075\%$$ | $$+0.002\%$$ | 1.77 | $$+0.034\%$$ | 0.040% |
| $$C_f$$, average over $$x \ge 0.1$$ | $$-0.868\%$$ | $$-0.564\%$$ | $$-0.481\%$$ | 1.88 | $$-0.450\%$$ | 0.039% |
| $$q_w$$, average over $$x \ge 0.1$$ | $$-0.614\%$$ | $$-0.303\%$$ | $$-0.210\%$$ | 1.74 | $$-0.170\%$$ | 0.050% |

The observed order is 1.56 to 1.88, close to the formal second order of the scheme. It is lower
than 2 because the leading-edge singularity affects these averages. At mid-plate, $$p = 2.01$$.

The GCI is below 0.08% in every row, so the fine-mesh results carry little mesh error. The mean
heat flux reaches the exact value within its GCI. For the other three quantities, $$E_0$$ is much
larger than the GCI, so the remaining difference is not mesh error. It comes from terms that the
Navier–Stokes solution contains and boundary-layer theory leaves out (of order
$$\mathrm{Re}_x^{-1/2}$$), or from the finite domain: the outlet is at the end of the plate and the
top boundary is at $$y = 0.2$$ m. The bend in the error near $$x = 1$$ in the figure below suggests
an outlet effect. A run on a larger domain would separate the two causes.

![Mesh convergence along the plate](/images/flatplate/flatplate_convergence.png)

*Error in $$C_f$$ and $$q_w$$ along the plate on the three meshes, labeled by cell count. The dotted
line is $$E_0$$ of the average over $$x \ge 0.1$$.*
{: .caption}

#### Summary

| Quantity | Criterion | Computed | Result |
|---|---|---|---|
| Skin friction error, $$x \geq 0.1$$ | $$\lvert E \rvert < 2\%$$ | $$+0.08\%$$ to $$-1.25\%$$ | pass |
| Wall heat flux error, $$x \geq 0.1$$ | $$\lvert E \rvert < 3\%$$ | $$-0.27\%$$ to $$-0.54\%$$ | pass |
| Velocity profile error, 5 stations | $$< 1\%$$ | $$0.32\%$$ | pass |
| Plate drag coefficient | — | $$4.2086 \times 10^{-3}$$ ($$+0.20\%$$) | |
| Mean wall heat flux | — | 130.58 W/m² ($$-0.07\%$$) | |

### Issues and possible improvements

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
- **Eleven silent exits.** Fluent closed with no message eleven times on this small 2D case, so
  each restart had to be checked against a known number.

#### Possible improvements

- Rerun on a larger domain, with the plate and the outlet extended past $$x = 1$$ and a higher top
  boundary, to separate the domain effect from the boundary-layer-theory terms.
- Rerun with real air properties at the film temperature (325 K) instead of the air-like values.
- Extract the velocity profiles on the finer mesh too. A mesh replacement there silently dropped
  the sampling lines.
- Next cases: variable properties and compressibility, then a turbulent flat plate.

## The exact solution

For a laminar layer with zero pressure gradient and constant fluid properties, the velocity
profiles at all stations have the same shape when the wall distance is scaled by the local layer
thickness. With the similarity variable

$$\eta = y\sqrt{\frac{U_\infty}{\nu x}}, \qquad \frac{u}{U_\infty} = f'(\eta), \qquad \theta(\eta) = \frac{T - T_w}{T_\infty - T_w},$$

the flow equations reduce to two ordinary differential equations, due to Blasius (velocity) and
Pohlhausen (temperature) [1, 3]:

$$f''' + \tfrac{1}{2} f f'' = 0, \qquad f(0) = f'(0) = 0, \quad f'(\infty) = 1,$$

$$\theta'' + \tfrac{1}{2}\,\mathrm{Pr}\, f\, \theta' = 0, \qquad \theta(0) = 0, \quad \theta(\infty) = 1.$$

I integrated both numerically, and recovered $$f''(0) = 0.332057$$ and, at
$$\mathrm{Pr} = 0.72$$, $$\theta'(0) = 0.295635$$. The first agrees with the tabulated value [2];
Blasius himself obtained 0.3317 by hand [1]. The wall values give skin friction and heat transfer along the
plate, with $$\mathrm{Re}_x = U_\infty x/\nu$$:

$$C_f = \frac{\tau_w}{\tfrac{1}{2}\rho U_\infty^2} = \frac{2 f''(0)}{\sqrt{\mathrm{Re}_x}} = \frac{0.664115}{\sqrt{\mathrm{Re}_x}}, \qquad \mathrm{Nu}_x = \frac{q_w\, x}{k\,(T_w - T_\infty)} = \theta'(0)\sqrt{\mathrm{Re}_x} = 0.295635\sqrt{\mathrm{Re}_x}.$$

The heat-transfer constant is for $$\mathrm{Pr} = 0.72$$, integrated here rather than taken from a
table. The common textbook correlation
$$0.332\,\mathrm{Pr}^{1/3}$$ gives 0.297565, which is 0.65% off. That is larger than the
discretization error measured below, so a correlation cannot serve as the reference here.

## References

1. H. Blasius, *Grenzschichten in Flüssigkeiten mit kleiner Reibung*, Zeitschrift für Mathematik
   und Physik **56** (1908), 1–37. English translation: *The Boundary Layers in Fluids with Little
   Friction*, NACA Technical Memorandum 1256, 1950. The velocity equation and its boundary
   conditions are on page 7 of the translation, in the scaling $$\zeta\zeta'' = -\zeta'''$$; the
   wall shear and the plate drag are on page 18.
2. L. Howarth, *On the solution of the laminar boundary layer equations*, Proceedings of the Royal
   Society A **164** (1938), 547–579. The accurate value $$f''(0) = 0.332057$$.
3. E. Pohlhausen, *Der Wärmeaustausch zwischen festen Körpern und Flüssigkeiten mit kleiner
   Reibung und kleiner Wärmeleitung*, ZAMM **1** (1921), 115–121. The thermal layer at a constant
   wall temperature.
4. ASME V&V 20-2009, *Standard for Verification and Validation in Computational Fluid Dynamics and
   Heat Transfer*. Grid convergence index.
5. L. Eça and M. Hoekstra, *Evaluation of numerical error estimation based on grid refinement
   studies with the method of the manufactured solutions*, Computers & Fluids **38** (2009),
   1580–1591.
6. NPARC Alliance CFD Verification and Validation Archive, *Laminar Flat Plate: Study #1*
   (case fplam01), NASA Glenn Research Center. The same case, used as a cross-check.
