---
title: "Laminar flat plate in Ansys Fluent: verification against Blasius"
excerpt: "<em>Work in progress.</em> Velocity profiles at five stations collapse onto the exact Blasius solution within 0.29%. Skin friction and wall heat flux are verified the same way on three meshes.<br/><img src='/images/flatplate/flatplate_similarity.png' width='400'>"
collection: portfolio
---

**Work in progress.** This page is not final. The text and the figures can change.
{: .notice--warning}

I set up, solved, and verified a steady laminar flat-plate boundary layer with heat transfer in
**Ansys Fluent**. The reference is the exact Blasius (velocity) and Pohlhausen (temperature)
similarity solution. On the finest of three meshes, skin friction is within 1.2% and wall heat
flux within 0.4% of the exact values over the judged range $$x \geq 0.1$$ m, and the observed
order of accuracy is close to the formal second order of the scheme.

## Setup

| Item | Value |
|---|---|
| Flow | $$U_\infty = 1$$ m/s, $$\mathrm{Re}_L = 10^5$$, plate length $$L = 1$$ m |
| Fluid | constant $$\rho$$ and $$\mu$$, $$\mathrm{Pr} = 0.72$$ (so Blasius is exact, not approximate) |
| Temperatures | freestream 300 K, isothermal wall 350 K |
| Domain | $$x \in [-0.5, 1.0]$$, $$y \in [0, 0.2]$$; symmetry plane ahead of the leading edge |
| Boundary conditions | velocity inlet; pressure outlets on the top and the exit; no-slip wall on the plate |
| Meshes | structured quadrilaterals, 2,700 / 10,800 / 43,200 cells, refinement ratio 2 |
| Solver | pressure-based, coupled, steady, laminar, energy on; second-order upwind |

The fluid properties are chosen so that $$\mathrm{Re}_x = x / 10^{-5}$$ exactly. The exact
wall values come from a numerical integration of the similarity equations,
$$f''(0) = 0.332058$$ and $$\theta'(0) = 0.295635$$ at $$\mathrm{Pr} = 0.72$$, not from a
correlation. The common correlation $$0.332\,\mathrm{Pr}^{1/3}$$ is off by 0.65%, which is larger
than the discretization error measured here.

## Flow field

![Streamwise velocity](/images/flatplate/flatplate_u.png)
*Streamwise velocity $$u$$, full domain to scale.*

![Gauge pressure](/images/flatplate/flatplate_p.png)
*Gauge pressure. It is flat everywhere except at the leading-edge peak, which is the zero
pressure-gradient assumption that the Blasius solution rests on.*

![Boundary layer close-up](/images/flatplate/flatplate_u_bl.png)
*The same velocity field near the wall, vertical scale $$\times 4$$. The layer grows from zero
thickness at $$x = 0$$ as $$\sqrt{x}$$.*

## Skin friction

![Skin friction](/images/flatplate/flatplate_cf.png)
*Top: $$C_f$$ on the finest mesh against Blasius. Middle and bottom: percent error, with the
$$\pm 2\%$$ band shaded. The large error near $$x = 0$$ is the $$x^{-1/2}$$ singularity of the
exact solution at the leading edge, not a mesh failure.*

## Wall heat flux

![Wall heat flux](/images/flatplate/flatplate_qw.png)
*Wall heat flux on the finest mesh against the exact $$65.339/\sqrt{x}$$ W/m², with the
$$\pm 3\%$$ band shaded. The error is about $$-0.2\%$$ and flat over the judged range.*

## Self-similarity

![Velocity profiles](/images/flatplate/flatplate_similarity.png)
*Velocity profiles at five stations, plotted against the similarity variable
$$\eta = y\sqrt{U_\infty/\nu x}$$. They collapse onto the Blasius profile within 0.29%.*

## Mesh convergence

![Mesh convergence](/images/flatplate/flatplate_convergence.png)
*Error in $$C_f$$ and $$q_w$$ on all three meshes. The dotted line is the Richardson-extrapolated
error as the mesh size goes to zero.*

The observed order of accuracy is 1.56 to 2.01, and the grid convergence index (GCI) is at most
0.077%. The extrapolated errors, $$-0.45\%$$ for $$C_f$$ and $$-0.17\%$$ for $$q_w$$, are much
larger than the GCI. The remaining difference is therefore resolved physics, not numerical error:
it is of order $$\mathrm{Re}_x^{-1/2}$$, the higher-order terms that boundary-layer theory leaves out.

## Summary

| Quantity | Criterion | Finest mesh | Result |
|---|---|---|---|
| Skin friction error, $$x \geq 0.1$$ | $$\lvert E \rvert < 2\%$$ | $$+0.20\%$$ to $$-1.15\%$$ | pass |
| Wall heat flux error, $$x \geq 0.1$$ | $$\lvert E \rvert < 3\%$$ | $$-0.13\%$$ to $$-0.36\%$$ | pass |
| Velocity profile error, 5 stations | $$< 1\%$$ | $$0.29\%$$ | pass |
| Plate drag coefficient | — | $$4.2136 \times 10^{-3}$$ ($$+0.32\%$$) | |
| Mean wall heat flux | — | 130.68 W/m² ($$+0.002\%$$) | |
