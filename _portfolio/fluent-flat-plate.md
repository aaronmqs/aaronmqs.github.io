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
in percent. The local errors in $$C_f$$ and $$q_w$$ are judged only for $$x \ge 0.1$$ m, because the
exact solution is singular at the leading edge and no mesh resolves it. The plate integrals
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
total heat transfer, per unit depth of the plate:

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

![Mesh convergence along the plate](/images/flatplate/flatplate_convergence.png)

*Error in $$C_f$$ and $$q_w$$ along the plate on the three meshes, labeled by cell count.*
{: .caption}

The error curves move less with each refinement, at close to second order. On the mesh used for
the results above, the remaining mesh error is below 0.2%.

One difference is still open. At mid-plate, the local $$C_f$$ converges to about 0.6% below
Blasius, and $$q_w$$ to about 0.2% below Pohlhausen. These differences are not mesh error. The
next test is a larger domain, with a higher top boundary and the outlet farther downstream of the
plate.

The drag coefficient $$C_d$$ behaves differently. Its error is $$-0.15\%$$, $$+0.20\%$$, and
$$+0.32\%$$ on the three meshes, and extrapolates to $$+0.38\%$$ on an infinitely fine mesh. The
local $$C_f$$ is below Blasius over most of the plate, but $$C_d$$ also includes the leading edge,
where the error is large and positive, and that part dominates the integral.

#### Summary

| Quantity | Criterion | Computed | Result |
|---|---|---|---|
| Skin friction error, $$x \geq 0.1$$ | $$\lvert E \rvert < 2\%$$ | $$+0.08\%$$ to $$-1.25\%$$ | pass |
| Wall heat flux error, $$x \geq 0.1$$ | $$\lvert E \rvert < 3\%$$ | $$-0.27\%$$ to $$-0.54\%$$ | pass |
| Velocity profile error, 5 stations | $$< 1\%$$ | $$0.32\%$$ | pass |
| Plate drag coefficient $$C_d$$ | — | $$4.2086 \times 10^{-3}$$ ($$+0.20\%$$; $$+0.38\%$$ extrapolated) | |
| Integrated wall heat flux $$Q$$ | — | 130.58 W/m ($$-0.08\%$$) | |

### Issues found

Several problems produced no error message. I caught each one by predicting a number and
checking it.

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

### Possible improvements

- Rerun on a larger domain, with the outlet farther downstream of the plate and a higher top
  boundary, to find the cause of the 0.6% difference in $$C_f$$ at mid-plate.
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
