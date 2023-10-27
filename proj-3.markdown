---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/data-acquisition-cleaning
---

## Data Acquisition and Cleaning

[< Prev](proj-2.markdown) &emsp; [Next >](proj-4.markdown)

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

```
def create_interpolator(df):
    x_values = sorted(df['x'].unique())
    Q_values = sorted(df['Q'].unique())
    f_values = df.set_index(['x', 'Q']).reindex(index=pd.MultiIndex.from_product([x_values, Q_values])).unstack().values
    return RegularGridInterpolator((x_values, Q_values), f_values, method='linear')
```

Here is a log plot that shows the momentum fraction times the PDFs, _x*f(x)_, as a funcion of _x_.  There are three types of partons plotted here: up quark, down quark, and gluon.

![PDFs Plot](https://raw.githubusercontent.com/reedhodges/portfolio_Jpsi/main/figures/pdfs-fig.png)