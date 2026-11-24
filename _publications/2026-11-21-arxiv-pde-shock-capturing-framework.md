---
title: "A principled framework for PDE-based shock capturing in high-order discontinuous Galerkin methods"
collection: publications
excerpt_separator: "<!--more-->"
category: manuscripts
permalink: /publication/2026-arxiv-pde-shock-capturing-framework
date: 2026-11-21
venue: 'arXiv preprint (submitted to the Journal of Computational Physics)'
paperurl: 'https://arxiv.org/abs/2611.28994'
citation: '<b>A. Marques</b>, A. H. Razavi, M. Yano, and M. J. Zahr, &quot;A principled framework for PDE-based shock capturing in high-order discontinuous Galerkin methods,&quot; arXiv preprint arXiv:2611.28994, 2026. Submitted to the <i>Journal of Computational Physics</i>.'
---
<!--more-->

## Abstract

PDE-based artificial viscosity (AV) captures shocks in high-order discontinuous Galerkin (DG) methods by solving an auxiliary PDE for a smooth viscosity field. The behavior of that field is set by how the terms of the AV PDE scale with the element size <i>h</i> and polynomial degree <i>p</i>, and by how its coefficients are parametrized. This work derives both choices from analysis and studies the impact these choices have on the implicit steady-state solver. First, an L<sup>2</sup>-projection analysis shows that the thinnest shock a polynomial space can represent monotonically scales with the resolution length <i>h&#772;</i> = <i>h</i>/<i>p</i>, and a scaling analysis of the AV PDE gives concrete conditions on its diffusion and source terms for the solution to be self-similar in <i>h&#772;</i>. Prior formulations satisfy them only under <i>h</i>-refinement at fixed <i>p</i>; the AV PDE is scaled so that they hold under <i>hp</i>-refinement, and the captured shock thins accordingly. Second, a Green's-function analysis of the steady AV PDE gives closed-form expressions for the width and magnitude of the AV. In the standard parametrization the two are coupled: raising the diffusion coefficient widens the AV but lowers its peak. An alternative parametrization of the decay coefficient removes the coupling in the Green's-function response and inverts it in the coupled state-AV problem, so that widening the AV raises its peak instead of lowering it. Third, a parameter study on a Mach 5 viscous flow over a half-cylinder characterizes how the choice of solution variables and the AV PDE's time, width, and magnitude coefficients affect the robustness and cost of the implicit solve. The choice of variables dominates robustness, with entropy variables converging over the largest region of the width-magnitude parameter space. Together, the three contributions make PDE-based AV more predictable to tune and its implicit solve more robust.
