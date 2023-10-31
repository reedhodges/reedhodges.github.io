---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/
---

<span style="color:red">**Disclaimer:**</span> This is a toy project, intended as a way for me to build and exemplify skills in Python and machine learning, and not intended to be a proper scientific study.  I utilize actual expressions for the cross sections derived by me and my collaborators in two papers [[1](https://arxiv.org/pdf/2308.08605),[2](https://arxiv.org/pdf/2310.13737)] published in 2023, but I make no claims of scientific rigor in the scikit-learn analysis.  Furthermore, this project is a work-in-progress and I will continue to make updates/improvements as I learn more and improve my skills.

## Outline

- [Introduction](proj-1.markdown)
- [Objectives](proj-2.markdown)
- [Data Acquisition and Cleaning](proj-3.markdown)
- [Exploratory Data Analysis](proj-4.markdown)
- [Model Development with scikit-learn](proj-6.markdown)
- [Results and Findings](proj-7.markdown)
- [Conclusion](proj-8.markdown)

## Introduction

The J/psi meson is a particle consisting of a charm quark and charm antiquark. It was discovered in 1974 by two research groups, one at the Stanford Linear Accelerator Center and the other at Brookhaven National Laboratory. 

One of the processes in which a J/psi can be produced is called semi-inclusive deep inelastic scattering. Here, an electron and a proton are collided at high speeds. A proton is not a fundamental particle - it has smaller constituent pieces inside it, called partons, which can be quarks or gluons. So when an electron collides with a proton, the proton can break apart and the partons can eventually produce even more types of particles, some of which might be the charm and anticharm quarks required to form a J/psi.  There are different ways this can happen, for example the process is different depending on whether the initial parton was a quark or a gluon.  My collaborators and I wrote a paper comparing these two possibilities.

Why might we be interested in this?  Well, clearly the problem depends on understanding the physics behind the partons inside the proton.  This information is encoded in *parton distribution functions*, or PDFs, which are like probability distributions that describe the likelihood to find a parton of a particular type and a particular momentum inside the proton. These PDFs are from experimental results. However, there are some types of PDFs we don't know much about yet.  As a theoretical physicist, something useful to do is to make a prediction for a future experiment, so that when the PDFs are measured, we can compare the results and test our understanding of the underlying physics.

The repository for this project can be found [here](https://github.com/reedhodges/portfolio_Jpsi).

---

[Next >](proj-2.markdown)