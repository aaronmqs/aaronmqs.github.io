---
title: "Laminar flat plate in Ansys Fluent: verification against Blasius"
excerpt: "Velocity profiles at five stations collapse onto the exact Blasius solution within 0.32%. Skin friction and wall heat flux are verified the same way on three meshes.<br/><img src='/images/flatplate/flatplate_fluent_u.png' width='400'>"
collection: portfolio
---

{% include toc title="Contents" %}

## The problem

I set up, solved, and verified a steady laminar flat-plate boundary layer with heat transfer in
**Ansys Fluent**.

**Result:** skin friction, wall heat flux, and velocity profiles all agree with the exact solution
within their tolerances (largest errors 1.25%, 0.54%, and 0.32%). A study on three meshes confirms
that the mesh error is below 0.2%.

The case is a steady laminar flow over an isothermal flat plate of length $$L = 1$$ m (Figure 1),
at a free-stream velocity of 1 m/s and $$\mathrm{Re}_L = 10^5$$. The wall is at 350 K in a 300 K stream. The fluid is
air-like: $$\mathrm{Pr} = 0.72$$, with constant density and viscosity, so the Blasius and
Pohlhausen similarity solution is the exact solution of the boundary-layer equations.

<div class="schematic-pair">
  <img src="/images/flatplate/flatplate_side.svg" alt="Side view of the flat plate: uniform flow from the left, boundary layer growing along the heated plate">
  <img src="/images/flatplate/flatplate_3d.svg" alt="Three-dimensional view of the flat plate with flow passing over its heated top surface">
</div>

*Side and three-dimensional views. The plate is drawn with thickness for visibility only.*
{: .caption}

The quantities of interest are the skin friction $$C_f$$, the wall heat flux $$q_w$$, and the
velocity profile across the boundary layer, all measured against the similarity solution. A
similar laminar flat-plate case, at Mach 0.1 and without heat transfer, is in the NPARC
archive [4].

![Fluent velocity contour](/images/flatplate/flatplate_fluent_u.png)

*Streamwise velocity computed in Fluent, near the leading edge (fine mesh, 43,200 cells). The
view shows about 5 mm ahead of the plate and the first 8 mm of the 1 m plate. The boundary layer starts at
the leading edge and grows along the plate.*
{: .caption}

## The computational model

### Domain and boundary conditions

Figure 3 shows the computational domain and its boundary conditions.

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

The mesh was built in **Ansys Meshing** (Workbench) and is structured quadrilaterals: 120 cells along the plate plus 60 ahead of it, times 60
normal to the wall, so $$(120 + 60) \times 60 = 10{,}800$$ cells. The first cell at the wall is
$$2.0 \times 10^{-4}$$ m high. Cells are clustered toward the wall and the leading edge.

### Setup in Fluent

| Item | Value |
|---|---|
| Software | Ansys Fluent 2025 R2, 2D, steady, laminar, energy equation on |
| Mesh | Ansys Meshing, mapped quadrilaterals, 10,800 cells |
| Flow | $$U_\infty = 1$$ m/s, $$\mathrm{Re}_L = 10^5$$, plate length $$L = 1$$ m |
| Fluid | air-like, constant properties: $$\rho = 1$$ kg/m³, $$\mu = 10^{-5}$$ kg/(m·s), $$\mathrm{Pr} = 0.72$$ |
| Temperatures | stream 300 K, isothermal wall 350 K |
| Domain | $$x \in [-0.5, 1.0]$$ m, $$y \in [0, 0.2]$$ m |
| Solver | pressure-based, coupled; second-order upwind for momentum and energy |

Real air at 300 K has $$\rho = 1.18$$ kg/m³ and $$\mu = 1.85 \times 10^{-5}$$ kg/(m·s), and both
change with temperature. Here they are rounded and held constant, which is the assumption of the
similarity solution. The comparison then contains only the discretization error, the terms that
boundary-layer theory omits, and the effect of the finite domain. The values also make
$$\mathrm{Re}_x = x/10^{-5}$$, with $$x$$ in meters. The Prandtl number is kept at the value for air, so the heat-transfer result applies to air.

### Solution and quantities of interest

The error in any computed quantity $$\phi$$ is $$E = (\phi - \phi_\mathrm{exact})/\phi_\mathrm{exact}$$,
in percent. The local errors in $$C_f$$ and $$q_w$$ are judged only for $$x \ge 0.1$$ m
($$\mathrm{Re}_x \ge 10^4$$). Near the leading edge, $$\mathrm{Re}_x$$ is small, so boundary-layer
theory does not apply, and its solution is singular at $$x = 0$$. The plate integrals
$$C_d$$ and $$Q$$ include the leading edge.

Each quantity has a pass tolerance, set in the test plan: 2% for $$C_f$$, 3% for $$q_w$$, and
1% for the velocity profiles.

In the figures below, each marker is a value that Fluent interpolates from the mesh nodes, and
each node value is an average of the surrounding cells. No marker is at a cell center.

#### Skin friction

The skin friction coefficient is the wall shear stress, made nondimensional with the dynamic
pressure of the stream:

$$C_f = \frac{\tau_w}{\tfrac{1}{2}\rho U_\infty^2}, \qquad \tau_w = \mu \left.\frac{\partial u}{\partial y}\right|_{y=0}.$$

The shear stress is the force per unit area that the fluid applies along the plate, so its
integral over the plate is the drag. The average of $$C_f$$ over the plate is the drag coefficient,
$$C_d = \frac{1}{L}\int_0^L C_f\,\mathrm{d}x$$. Because $$C_f$$ depends on the velocity gradient at the wall, its error shows if the cells next to the
wall are small enough.

Near the leading edge the error is large,
up to $$+50\%$$ at the first node. It comes from the $$x^{-1/2}$$ singularity of the exact
solution, not from a mesh failure. For $$x \ge 0.1$$, the error is between $$+0.08\%$$ and
$$-1.25\%$$. The largest errors are at the trailing edge, where the outlet is at the end of the
plate.

![Skin friction](/images/flatplate/flatplate_cf.png)

*Top: $$C_f$$ against Blasius. Bottom: error in percent, with the $$\pm 2\%$$ band shaded. The
first two points ($$+50\%$$ and $$+17\%$$) are above the range of the bottom plot.*
{: .caption}

#### Wall heat flux

The wall heat flux is the heat per unit area that the plate gives to the fluid, by conduction
through the fluid at the wall:

$$q_w = -k \left.\frac{\partial T}{\partial y}\right|_{y=0}.$$

It is the cooling rate, the number an engineer designs with. Its integral over the plate is the
integrated wall heat flux, the heat rate per unit depth of the plate:

$$Q = \int_0^L q_w\,\mathrm{d}x \quad \mathrm{W/m}.$$

Because $$q_w$$ depends on the temperature gradient at the wall, it tests the energy equation.
With the Pohlhausen solution, the exact wall heat flux is

$$q_w = k\,(T_w - T_\infty)\,\theta'(0)\sqrt{\frac{U_\infty}{\nu x}} = \frac{65.339}{\sqrt{x}}\ \mathrm{W/m^2},$$

with $$k = 0.013978$$ W/(m·K), the value that gives $$\mathrm{Pr} = 0.72$$ with Fluent's
$$c_p = 1006.43$$ J/(kg·K). Its integral is $$Q = 2 \times 65.339\sqrt{L} = 130.68$$ W/m.

For $$x \ge 0.1$$, the error is
between $$-0.27\%$$ and $$-0.54\%$$.

![Wall heat flux](/images/flatplate/flatplate_qw.png)

*Top: $$q_w$$ against the exact solution. Bottom: error in percent, with the $$\pm 3\%$$ band
shaded. The first point ($$+48\%$$) is above the range of the bottom plot.*
{: .caption}

#### Velocity profiles

Wall values test only the first cells. The profiles test the whole layer.

The velocity is divided by the local edge velocity $$U_e$$, so the comparison tests the shape of
the profile. Each marker is a point where a vertical sampling line crosses a mesh line. The
largest error is
$$0.32\%$$.

![Velocity profiles](/images/flatplate/flatplate_similarity.png)

*Top: velocity profiles at five stations against the Blasius profile $$f'(\eta)$$. Bottom: error
$$u/U_e - f'(\eta)$$ in percent, with the $$\pm 1\%$$ band shaded.*
{: .caption}

#### Mesh convergence

A match with the exact solution on one mesh can be luck, so the case was also solved on a coarser
mesh (2,700 cells) and a finer mesh (43,200 cells), with half and twice the cells in each direction
and the same grading.

**The mesh error goes to zero.** The mesh error is the difference between the value on a mesh and
the value extrapolated to an infinitely fine mesh (Richardson extrapolation from the three meshes):

| Quantity | 2,700 cells | 10,800 cells | 43,200 cells | Measured order |
|---|---|---|---|---|
| $$C_f$$ at $$x = 0.5$$ | $$-0.36\%$$ | $$-0.09\%$$ | $$-0.02\%$$ | 2.0 |
| $$q_w$$ at $$x = 0.5$$ | $$-0.37\%$$ | $$-0.11\%$$ | $$-0.03\%$$ | 1.8 |
| $$C_d$$ | $$-0.53\%$$ | $$-0.18\%$$ | $$-0.06\%$$ | 1.6 |
| $$Q$$ | $$-0.37\%$$ | $$-0.11\%$$ | $$-0.03\%$$ | 1.8 |

*How the mesh error is computed.* Each refinement halves the cell size. Take $$C_f$$ at
$$x = 0.5$$, as the error against Blasius in percent: $$-0.972$$ on the coarse mesh, $$-0.701$$ on
the medium mesh, $$-0.634$$ on the fine mesh.

1. **Steps between meshes:** coarse to medium $$+0.271$$, medium to fine $$+0.067$$.
2. **Order:** the step shrinks by $$0.271 / 0.067 = 4.0 = 2^2$$, so the order is $$p = 2$$.
3. **Remaining steps:** if each next step is 4 times smaller, the steps after the fine mesh add up
   to $$0.067 \times (\tfrac{1}{4} + \tfrac{1}{16} + \dots) = 0.067 / 3 = 0.022$$.
4. **Infinitely fine mesh:** $$-0.634 + 0.022 = -0.612$$.
5. **Mesh error** on each mesh: its value minus $$(-0.612)$$, so $$-0.36$$, $$-0.09$$, and
   $$-0.02$$, as in the table.

This is Richardson extrapolation. The other rows are computed the same way, each with its own
measured order.

Each refinement divides the mesh error by 3 to 4, close to the second order of the scheme. On the
mesh used for the results above (10,800 cells), the mesh error is below 0.2%.

**But Fluent and Blasius solve two different problems.** Fluent solves the full Navier–Stokes
equations in a finite domain. Blasius and Pohlhausen solve the boundary-layer equations for a
semi-infinite plate in an unbounded stream with zero pressure gradient. So, on an infinitely fine
mesh, the error against the exact solution does not go to zero. It goes to the difference between
the two problems:

| Quantity | Error against the exact solution, infinitely fine mesh |
|---|---|
| $$C_f$$ at $$x = 0.5$$ | $$-0.61\%$$ |
| $$q_w$$ at $$x = 0.5$$ | $$-0.20\%$$ |
| $$C_d$$ | $$+0.38\%$$ |
| $$Q$$ | $$+0.03\%$$ |

![Mesh convergence along the plate](/images/flatplate/flatplate_convergence.png)

*Error in $$C_f$$ and $$q_w$$ against the exact solution along the plate, on the three meshes,
labeled by cell count. The curves converge, but not to zero.*
{: .caption}

![Mesh convergence of the plate integrals](/images/flatplate/flatplate_integrated.png)

*Error in $$C_d$$ and $$Q$$ against the exact solution, on the three meshes, with the fit used for
the extrapolation. For $$C_d$$, the error grows with refinement: on the coarse mesh, the mesh error
and the difference between the two problems almost cancel.*
{: .caption}

Two differences between the problems explain these values:

- **The leading edge.** Near the leading edge, $$\mathrm{Re}_x$$ is small and boundary-layer theory
  does not apply. The plate integrals $$C_d$$ and $$Q$$ include this region, the local values at
  $$x = 0.5$$ do not.
- **The finite domain.** The top boundary is only 0.2 m above the plate, and the outlet is at the
  trailing edge. These boundaries change the flow outside the boundary layer a little, and Blasius
  assumes an unbounded stream. This is the probable cause of the $$-0.6\%$$ in $$C_f$$. A larger
  domain would test this.

### Lessons learned

- **Convergence.** The default residual criteria stopped the solver too early for this case. I
  tightened them and also monitored the drag coefficient until its value stopped changing.
- **Mesh controls.** In Ansys Meshing, *Blend to Neighbors* is on by default for each edge sizing,
  and it can change the requested number of divisions. I turned it off to keep the exact cell
  counts of the structured mesh.
- **First-cell height.** The *Bias Factor* sets the ratio of the largest to the smallest cell, not
  the first-cell height. I checked the first-cell height directly from the mesh.

### Possible improvements

- Rerun on a larger domain, with the outlet farther downstream of the plate and a higher top
  boundary, to confirm that the finite domain causes the 0.6% difference in $$C_f$$ at
  mid-plate.
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
Blasius himself obtained 0.3317 by hand [1]. The wall values give skin friction and heat transfer
along the plate, with $$\mathrm{Re}_x = U_\infty x/\nu$$:

$$C_f = \frac{\tau_w}{\tfrac{1}{2}\rho U_\infty^2} = \frac{2 f''(0)}{\sqrt{\mathrm{Re}_x}} = \frac{0.664115}{\sqrt{\mathrm{Re}_x}}, \qquad \mathrm{Nu}_x = \frac{q_w\, x}{k\,(T_w - T_\infty)} = \theta'(0)\sqrt{\mathrm{Re}_x} = 0.295635\sqrt{\mathrm{Re}_x}.$$

The heat-transfer constant is for $$\mathrm{Pr} = 0.72$$, integrated here rather than taken from a
table. The common textbook correlation
$$0.332\,\mathrm{Pr}^{1/3}$$ gives 0.297565, which is 0.65% off. That is larger than the
discretization error measured above, so a correlation cannot serve as the reference here.

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
4. NPARC Alliance CFD Verification and Validation Archive, *Laminar Flat Plate: Study #1*
   (case fplam01), NASA Glenn Research Center. Mach 0.1, $$\mathrm{Re}_L \approx 2 \times 10^5$$,
   no heat transfer, compared with Blasius.
