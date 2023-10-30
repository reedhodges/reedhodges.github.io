---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/results
---

## Results and Findings

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

---

[< Prev](proj-6.markdown) &emsp; [Next >](proj-8.markdown)