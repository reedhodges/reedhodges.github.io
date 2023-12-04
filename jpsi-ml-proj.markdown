---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/
---

<span style="color:red">**Disclaimer:**</span> This is a toy project, intended as a way for me to build and exemplify skills in Python and machine learning, and not intended to be a proper scientific study.  I utilize actual expressions for the cross sections derived by me and my collaborators in two papers [[1](https://arxiv.org/pdf/2308.08605),[2](https://arxiv.org/pdf/2310.13737)] published in 2023, but I make no claims of scientific rigor in the scikit-learn analysis.  Furthermore, this project is a work-in-progress and I will continue to make updates/improvements as I learn more and improve my skills.

## <ins>Introduction</ins>

The J/psi meson is a particle consisting of a charm quark and charm antiquark. It was discovered in 1974 by two research groups, one at the Stanford Linear Accelerator Center and the other at Brookhaven National Laboratory. 

One of the processes in which a J/psi can be produced is called semi-inclusive deep inelastic scattering. Here, an electron and a proton are collided at high speeds. A proton is not a fundamental particle - it has smaller constituent pieces inside it, called partons, which can be quarks or gluons. So when an electron collides with a proton, the proton can break apart and the partons can eventually produce even more types of particles, some of which might be the charm and anticharm quarks required to form a J/psi.  There are different ways this can happen, for example the process is different depending on whether the initial parton was a quark or a gluon.  My collaborators and I wrote a paper comparing these two possibilities.

Why might we be interested in this?  Well, clearly the problem depends on understanding the physics behind the partons inside the proton.  This information is encoded in *parton distribution functions*, or PDFs, which are like probability distributions that describe the likelihood to find a parton of a particular type and a particular momentum inside the proton. These PDFs are from experimental results. However, there are some types of PDFs we don't know much about yet.  As a theoretical physicist, something useful to do is to make a prediction for a future experiment, so that when the PDFs are measured, we can compare the results and test our understanding of the underlying physics.

The repository for this project can be found [here](https://github.com/reedhodges/portfolio_Jpsi).

## <ins>Project Objectives</ins>

### Overview

In the fascinating realm of particle physics, the concept of phase space and its respective cross sections are crucial for understanding particle interactions. The objective of this project is to harness the power of data science and machine learning to learn more about the kinematic parameters for J/psi production cross sections.

### Specific Goals

1. **Data Collection and Preprocessing:** Acquire a comprehensive dataset that encapsulates key variables and parameters that influence the cross sections.
2. **Feature Identification:** Pinpoint the most salient features that have a potential impact on the magnitude of the cross sections.
3. **Predictive Modeling:** Identify a machine learning model capable of accurately predicting regions of phase space where the cross sections are large.
4. **Model Validation:** Ensure the model is both accurate and robust through rigorous testing and validation.

### Challenges

- **Data Complexity:** Phase space data is inherently multi-dimensional and can be challenging to process and understand.
- **High Variability:** Due to the quantum nature of particle interactions and experimental error, there is variability and uncertainty inherent in the data.
- **Computational Demands:** The high granularity of the data requires efficient computational methods to process and analyze.

### Significance

Understanding regions of phase space where cross sections are significant has wide-ranging implications:

- **Particle Collider Experiments:** Helps in predicting outcomes and planning experiments in particle colliders.
- **Testing Theory:** Assists researchers in validating or refuting theoretical models.
- **Advancing Physics:** Understanding particle interactions can lead to new discoveries.

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

## <ins>Exploratory Data Analysis</ins>

The initial exploratory data analysis for this project was done in a [paper](https://arxiv.org/pdf/2310.13737) published by me and my collaborators.  Look at Figure 4 from that paper:

![PDFs Plot](https://raw.githubusercontent.com/reedhodges/reedhodges.github.io/main/expl_data_analysis.png)

Here, we are plotting the cross section (\[sigma]) differential in the transverse momentum of the J/psi, as a function of that transverse momentum.  Each plot is a bin where we integrated over a particular subset of the domain of *x*, *z*, and *Q*.  The blue lines are where the J/psi is produced from a quark inside the proton, and the red and green lines are two different ways the J/psi can be produced from a gluon inside the proton.  The purpose of making this plot was to identify bins in which one of these lines was dominant over the other.  For example, in the third bin the red and green lines are clearly dominant over the blue, while in the bottom row of plots the blue line is dominant.  Some conclusions you can draw from this include:

- If you are interested in production via a gluon, look at larger values of *z*, but smaller values of *x* and *Q*
- If you are interested in production via a quark, look at larger values of *x*

## <ins>Feature Engineering and Selection</ins>
**in progress**


## <ins>Model Development with scikit-learn</ins>

### Model Choice: Random Forest Regressor

For this project, we've chosen the `RandomForestRegressor` from the Scikit-learn library. The Random Forest algorithm is an ensemble learning method, which combines multiple decision trees to produce a more accurate and robust model. It is well-suited for our dataset because:

- It can handle high-dimensional data efficiently.
- It provides a natural resistance to overfitting.
- It offers insights into feature importance, allowing for a better understanding of our data.

### Model Training

The dataset was split into training and test sets, with 80% of the data used for training and 20% reserved for testing.

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

The model parameters were initialized as follows:

- number of estimators (trees): 100
- maximum depth of the tree: 10

These parameters were chosen to balance the accuracy and computational efficiency.

The model was then trained on the data.

```python
rf = RandomForestRegressor(n_estimators=100, max_depth=10, random_state=42)
rf.fit(X_train, y_train.values.ravel())
```

## <ins>Results and Findings</ins>

### Model Performance

Our trained Random Forest model exhibited solid performance in predicting the regions in phase space. The performance was evaluated using the Mean Squared Error (MSE) for both the training and test data. The results were:

- **Training MSE:** 0.0005671118252182426
- **Test MSE:** 0.0005891680348386154

The close values of MSE for both training and test datasets indicate that our model generalizes well to unseen data, without any significant signs of overfitting.

### Feature Importances

The Random Forest Regressor offers insights into the importance of features used for predictions. The feature importances from our model were:

- **x:** 0.6807263601754133
- **z:** 0.006080546460394571
- **Q:** 0.19588435868521697
- **PT:** 0.11730873467897507

From the above, we can deduce:

- **x** plays a dominant role in determining the regions in phase space with a significance of approximately 68.07%.
- **Q** and **PT** also have considerable importances with around 19.59% and 11.73% respectively.
- **z**, on the other hand, appears to have a minor role with an importance of just about 0.61%.

Recall that, in the appropriate references frames, *x* represents the fraction of the proton's momentum carried by the struck quark, *z* denotes the ratio of the energy of the J/psi to the energy of the virtual photon, and *Q* stands for the momentum transfer's scale or the virtual photon's virtuality. The significant importance of *x* in our findings highlights the dominant role the struck quark's momentum fraction plays in determining the phase space regions. Collectively, understanding the significance and interplay of these parameters can provide a richer comprehension of QCD phenomena and the underlying mechanisms of hadronic interactions.

## <ins>Conclusion</ins>

### Reflecting on Objectives

We embarked on this project with the aim of learning about the kinematic parameters of J/psi production cross sections using machine learning. Our model offered insights into the importance of these parameters in semi-inclusive deep inelastic scattering.

### Key Takeaways

- Our **Random Forest Regressor** model delivered commendable performance, demonstrating its potential in predicting complex particle interactions.
  
- The parameter *x*, representing the fraction of the proton's momentum carried by the struck quark, emerged as the dominant factor in our findings. This reaffirms its centrality in deep inelastic scattering events.

- Other parameters, such as *Q* and *PT*, also displayed notable importance.

- The minor role of *z* in our results, despite its significance in the theoretical framework, hints at the potential need for further investigations.

### Future Directions

There is always room for improvement and further exploration:

1. **Model Enhancement:** Experimenting with other machine learning algorithms or deep learning models might yield even better results.
   
2. **Expanded Dataset:** Incorporating a broader range of data, perhaps with PDFs from different experiments, could enhance the model's robustness and generalization.

3. **Feature Engineering:** Further research into creating new features or refining existing ones based on theoretical insights might lead to better predictive capabilities.

In conclusion, this project underscores the use of machine learning in the realm of nuclear/particle physics. It not only aids in prediction but also works towards a deeper understanding of these complex phenomena.