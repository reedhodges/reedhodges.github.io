---
layout: page
title: J/ψ production ML project
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

<p>The J/ψ meson is a particle consisting of a charm quark and charm antiquark. It was discovered in 1974 by two research groups, one at the Stanford Linear Accelerator Center and the other at Brookhaven National Laboratory. </p>

<p>One of the processes in which a J/ψ can be produced is called semi-inclusive deep inelastic scattering. Here, an electron and a proton are collided at high speeds. A proton is not a fundamental particle - it has smaller constituent pieces inside it, called partons, which can be quarks or gluons. So when an electron collides with a proton, the proton can break apart and the partons can eventually produce even more types of particles, some of which might be the charm and anticharm quarks required to form a J/ψ.  There are different ways this can happen, for example the process is different depending on whether the initial parton was a quark or a gluon.  My collaborators and I wrote a paper comparing these two possibilities.</p>

<p>Why might we be interested in this?  Well, clearly the problem depends on understanding the physics behind the partons inside the proton.  This information is encoded in <em>parton distribution functions</em>, or PDFs, which are like probability distributions that describe the likelihood to find a parton of a particular type and a particular momentum inside the proton. These PDFs are from experimental results. However, there are some types of PDFs we don't know much about yet.  As a theoretical physicist, something useful to do is to make a prediction for a future experiment, so that when the PDFs are measured, we can compare the results and test our understanding of the underlying physics.</p>

<h3>Project Objectives</h3>

<p>This portfolio project focuses on the application of machine learning techniques to a specialized domain of particle physics, specifically analyzing J/ψ production cross sections. The dataset comprises several measurements of J/ψ production cross sections as a function of four kinematic variables. The primary goal is to leverage Python's machine learning ecosystem to extract meaningful insights and predictions from this complex dataset.</p>

<h4>Specific Objectives</h4>

<h5>1. Data Visualization and Analysis</h5>
<ul>
    <li><strong>Goal:</strong> Conduct a comprehensive exploratory data analysis to uncover patterns, correlations, and distributions within the data.</li>
    <li><strong>Methodology:</strong> Employ Python's visualization libraries to create insightful visualizations that inform subsequent modeling choices and provide an intuitive understanding of the dataset's characteristics.</li>
</ul>

<h5>2. Predictive Modeling</h5>
<ul>
    <li><strong>Goal:</strong> Develop and train machine learning models to accurately predict the J/ψ production cross section based on the provided kinematic variables.</li>
    <li><strong>Methodology:</strong> Experiment with various regression models to determine the most effective approach for this dataset.</li>
</ul>

<h5>3. Feature Importance Analysis</h5>
<ul>
    <li><strong>Goal:</strong> Identify and quantify the influence of each kinematic variable on the J/ψ production cross section.</li>
    <li><strong>Methodology:</strong> Utilize feature importance metrics from machine learning models and techniques like SHAP values to understand the contribution of each variable to the model's predictions.</li>
</ul>

<h5>4. Anomaly Detection</h5>
<ul>
    <li><strong>Goal:</strong> Detect and analyze anomalies in the J/ψ production cross section data to ensure the integrity and quality of the machine learning models.</li>
    <li><strong>Methodology:</strong> Implement unsupervised learning algorithms to identify outliers and unusual patterns in the dataset, aiding in data cleaning and preprocessing.</li>
</ul>

<h4>Expected Outcomes</h4>
<p>By accomplishing these objectives, this project aims to demonstrate a proficient application of machine learning in a physics context, showcasing skills in data processing, model development, and analysis. The project will also emphasize the importance of exploratory data analysis and feature understanding in building effective machine learning models. The final outcome will not only include a set of trained models but also a rich set of visualizations and insights into the J/ψ production proces</p>

<p>Through predictive modeling, we anticipate establishing a reliable and accurate relationship between the kinematic variables and the J/ψ production cross section. The feature importance analysis will provide a deeper understanding of the driving factors in J/ψ production, which could have broader implications in theoretical and experimental physics. The exploratory data analysis will serve as a cornerstone for all subsequent modeling, ensuring a robust and well-informed approach. Finally, the anomaly detection component will enhance the dataset's quality, ensuring the reliability of our models and findings.</p>

  </div>
</div>

<div class="collapse" id="dataset">
  <div class="card card-body">


<h3>PDFs</h3>

<p>The most fundamental type of data we will be using are derived from <em>parton distribution functions</em>, or PDFs. These are probability distributions that are a function of the fraction <em>x</em> of the proton's momentum carried by the parton, and a particular energy scale in the problem <em>Q</em>. In practice, the PDFs are obtained by fitting to experimental data, which is done by different collaborations of scientists. The collaborations then publish their fits for use by other researchers.</p>

<p>The standard tool for evaluating PDFs is <a href="https://lhapdf.hepforge.org/">LHAPDF</a>, which has a library of the fits done by various collaborations. <a href="https://ncteq.hepforge.org/mma/index.html">ManeParse</a> and <a href="https://github.com/prokudin/WW-SIDIS">WW-SIDIS</a> provide Mathematica interfaces to evaluate the PDFs, which was of particular utility to me and my collaborators in writing our research papers, as much of our calculations and plotting were done in Mathematica.</p>

<p>Once those packages are installed, generating <code>.csv</code> files of PDF data is incredibly straightforward. The code below generates the gluon PDF from ManeParse and the up and down quark PDFs from WW-SIDIS.</p>

<pre><code class="wolfram">
gPDF=Flatten[Table[{x,Q,pdfFunction[5,0,x,Q]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
uPDF=Flatten[Table[{x,Q,f1u[x,Q^2]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
dPDF=Flatten[Table[{x,Q,f1d[x,Q^2]},{x,0.01,1.0,(1.0-0.01)/200},{Q,5,50,(50.0-5.0)/200}],1]
Export[NotebookDirectory[]<>"pdf_data/gPDF.csv",gPDF]
Export[NotebookDirectory[]<>"pdf_data/uPDF.csv",uPDF]
Export[NotebookDirectory[]<>"pdf_data/dPDF.csv",dPDF]
</code></pre>

<p>Each PDF is evaluated for 200<sup>2</sup> = 40,000 points. The <code>.csv</code> files can be imported as a pandas DataFrame, and we can use <code>scipy</code> to create an interpolator for other values of <em>x</em> and <em>Q</em>.</p>

<pre><code class="python">
function create_interpolator(df):
    x_values = sorted(df['x'].unique())
    Q_values = sorted(df['Q'].unique())
    f_values = df.set_index(['x', 'Q']).reindex(index=pd.MultiIndex.from_product([x_values, Q_values])).unstack().values
    return RegularGridInterpolator((x_values, Q_values), f_values, method='linear')
</code></pre>

<p>Here is a log plot that shows the momentum fraction times the PDFs, <em>x*f(x)</em>, as a function of <em>x</em>. There are three types of partons plotted here: up quark, down quark, and gluon.</p>

<img src="https://raw.githubusercontent.com/reedhodges/portfolio_Jpsi/main/figures/pdfs-fig.png" alt="PDFs Plot">

  </div>
</div>

<div class="collapse" id="EDA">
  <div class="card card-body">

<p>The initial exploratory data analysis for this project was done in a <a href="https://arxiv.org/pdf/2310.13737">paper</a> published by me and my collaborators. Look at Figure 4 from that paper:</p>

<img src="https://raw.githubusercontent.com/reedhodges/reedhodges.github.io/main/expl_data_analysis.png" alt="PDFs Plot">

<p>Here, we are plotting the cross section (<code>&sigma;</code>) differential in the transverse momentum of the J/ψ, as a function of that transverse momentum. Each plot is a bin where we integrated over a particular subset of the domain of <em>x</em>, <em>z</em>, and <em>Q</em>. The blue lines are where the J/ψ is produced from a quark inside the proton, and the red and green lines are two different ways the J/ψ can be produced from a gluon inside the proton. The purpose of making this plot was to identify bins in which one of these lines was dominant over the other. For example, in the third bin, the red and green lines are clearly dominant over the blue, while in the bottom row of plots the blue line is dominant. Some conclusions you can draw from this include:</p>

<ul>
  <li>If you are interested in production via a gluon, look at larger values of <em>z</em>, but smaller values of <em>x</em> and <em>Q</em></li>
  <li>If you are interested in production via a quark, look at larger values of <em>x</em></li>
</ul>

  </div>
</div>
