---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/
---

<span style="color:red">**Disclaimer:**</span> This is a toy project, intended as a way for me to build and exemplify skills in Python and machine learning, and not intended to be a proper scientific study.  I utilize actual expressions for the cross sections derived by me and my collaborators in two papers [[1](https://arxiv.org/pdf/2308.08605),[2](https://arxiv.org/pdf/2310.13737)] published in 2023, but I make no claims of scientific rigor in the scikit-learn analysis.  Furthermore, this project is a work-in-progress and I will continue to make updates/improvements as I learn more and improve my skills.

The repository for this project can be found [here](https://github.com/reedhodges/portfolio_Jpsi).

<div class="btn-group" role="group" aria-label="button group">
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#introduction" aria-expanded="false" aria-controls="introduction">
    Introduction
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#dataset" aria-expanded="false" aria-controls="dataset">
    Dataset
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#EDA" aria-expanded="false" aria-controls="EDA">
    Exploratory data analysis
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#predictive-modeling" aria-expanded="false" aria-controls="predictive-modeling">
    Predicting modeling
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#feature-importance" aria-expanded="false" aria-controls="feature-importance">
    Feature importance
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#anomaly-detection" aria-expanded="false" aria-controls="anomaly-detection">
    Anomaly detection
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#conclusion" aria-expanded="false" aria-controls="conclusion">
    Conclusion
  </button>
</div>


<div class="collapse" id="introduction">
  <div class="card card-body">

The J/psi meson is a particle consisting of a charm quark and charm antiquark. It was discovered in 1974 by two research groups, one at the Stanford Linear Accelerator Center and the other at Brookhaven National Laboratory. 
<br>
One of the processes in which a J/psi can be produced is called semi-inclusive deep inelastic scattering. Here, an electron and a proton are collided at high speeds. A proton is not a fundamental particle - it has smaller constituent pieces inside it, called partons, which can be quarks or gluons. So when an electron collides with a proton, the proton can break apart and the partons can eventually produce even more types of particles, some of which might be the charm and anticharm quarks required to form a J/psi.  There are different ways this can happen, for example the process is different depending on whether the initial parton was a quark or a gluon.  My collaborators and I wrote a paper comparing these two possibilities.
<br>
Why might we be interested in this?  Well, clearly the problem depends on understanding the physics behind the partons inside the proton.  This information is encoded in *parton distribution functions*, or PDFs, which are like probability distributions that describe the likelihood to find a parton of a particular type and a particular momentum inside the proton. These PDFs are from experimental results. However, there are some types of PDFs we don't know much about yet.  As a theoretical physicist, something useful to do is to make a prediction for a future experiment, so that when the PDFs are measured, we can compare the results and test our understanding of the underlying physics.

<h1>Project Objectives</h1>

<h2>Overview</h2>

In the fascinating realm of particle physics, the concept of phase space and its respective cross sections are crucial for understanding particle interactions. The objective of this project is to harness the power of data science and machine learning to learn more about the kinematic parameters for J/psi production cross sections.

<h2>Specific goals</h2>

1. **Data Collection and Preprocessing:** Acquire a comprehensive dataset that encapsulates key variables and parameters that influence the cross sections.
2. **Feature Identification:** Pinpoint the most salient features that have a potential impact on the magnitude of the cross sections.
3. **Predictive Modeling:** Identify a machine learning model capable of accurately predicting regions of phase space where the cross sections are large.
4. **Model Validation:** Ensure the model is both accurate and robust through rigorous testing and validation.

<h2>Challenges</h2>

- **Data Complexity:** Phase space data is inherently multi-dimensional and can be challenging to process and understand.
- **High Variability:** Due to the quantum nature of particle interactions and experimental error, there is variability and uncertainty inherent in the data.
- **Computational Demands:** The high granularity of the data requires efficient computational methods to process and analyze.

<h2>Significance</h2>

Understanding regions of phase space where cross sections are significant has wide-ranging implications:

- **Particle Collider Experiments:** Helps in predicting outcomes and planning experiments in particle colliders.
- **Testing Theory:** Assists researchers in validating or refuting theoretical models.
- **Advancing Physics:** Understanding particle interactions can lead to new discoveries.

  </div>
</div>

<div class="collapse" id="dataset">
  <div class="card card-body">

## <ins>Data Acquisition and Cleaning</ins>

# PDFs

The most fundamental type of data we will be using are derived from *parton distribution functions*, or PDFs.  These are probability distributions that are a function of the fraction *x* of the proton's momentum carried by the parton, and a particular energy scale in the problem *Q*.  In practice, the PDFs are obtained by fitting to experimental data, which is done by different collaborations of scientists.  The collaborations then publish their fits for use by other researchers.  

The standard tool for evaluating PDFs is [LHAPDF](https://lhapdf.hepforge.org/), which has a library of the fits done by various collaborations.  [ManeParse](https://ncteq.hepforge.org/mma/index.html) and [WW-SIDIS](https://github.com/prokudin/WW-SIDIS) provide Mathematica interfaces to evaluate the PDFs, which was of particular utility to me and my collaborators in writing our research papers, as much of our calculations and plotting were done in Mathematica.

Once those packages are installed, generating `.csv` files of PDF data is incredibly straightforward.  The code below generates the gluon PDF from ManeParse and the up and down quark PDFs from WW-SIDIS.

```
gPDF=Flatten[Table[{x,Q,pdfFunction[5,0,x,Q]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
uPDF=Flatten[Table[{x,Q,f1u[x,Q^2]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
dPDF=Flatten[Table[{x,Q,f1d[x,Q^2]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
Export[NotebookDirectory[]<>"pdf_data/gPDF.csv",gPDF]
Export[NotebookDirectory[]<>"pdf_data/uPDF.csv",uPDF]
Export[NotebookDirectory[]<>"pdf_data/dPDF.csv",dPDF]
```

Each PDF is evaluated for 200<sup>2</sup> = 40,000 points.  The `.csv` files can be imported as a pandas DataFrame, and we can use `scipy` to create an interpolator for other values of *x* and *Q*.

```python
def create_interpolator(df):
    x_values = sorted(df['x'].unique())
    Q_values = sorted(df['Q'].unique())
    f_values = df.set_index(['x', 'Q']).reindex(index=pd.MultiIndex.from_product([x_values, Q_values])).unstack().values
    return RegularGridInterpolator((x_values, Q_values), f_values, method='linear')
```

Here is a log plot that shows the momentum fraction times the PDFs, _x*f(x)_, as a funcion of _x_.  There are three types of partons plotted here: up quark, down quark, and gluon.

![PDFs Plot](https://raw.githubusercontent.com/reedhodges/portfolio_Jpsi/main/figures/pdfs-fig.png)

  </div>
</div>

<div class="collapse" id="EDA">
  <div class="card card-body">

## <ins>Exploratory Data Analysis</ins>

The initial exploratory data analysis for this project was done in a [paper](https://arxiv.org/pdf/2310.13737) published by me and my collaborators.  Look at Figure 4 from that paper:

![PDFs Plot](https://raw.githubusercontent.com/reedhodges/reedhodges.github.io/main/expl_data_analysis.png)

Here, we are plotting the cross section (\[sigma]) differential in the transverse momentum of the J/psi, as a function of that transverse momentum.  Each plot is a bin where we integrated over a particular subset of the domain of *x*, *z*, and *Q*.  The blue lines are where the J/psi is produced from a quark inside the proton, and the red and green lines are two different ways the J/psi can be produced from a gluon inside the proton.  The purpose of making this plot was to identify bins in which one of these lines was dominant over the other.  For example, in the third bin the red and green lines are clearly dominant over the blue, while in the bottom row of plots the blue line is dominant.  Some conclusions you can draw from this include:

- If you are interested in production via a gluon, look at larger values of *z*, but smaller values of *x* and *Q*
- If you are interested in production via a quark, look at larger values of *x*

  </div>
</div>
