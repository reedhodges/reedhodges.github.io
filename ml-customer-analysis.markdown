---
layout: page
title: Machine learning for customer analysis
permalink: /ml-customer-analysis/
---

Recently I wanted to learn how to run SQL queries.  In deciding which data to use, I wanted a dataset where I'd have some level of intuition for what the data points mean.  An obvious choice is sports statistics, and basketball is the sport with which I'm most familiar.  

I decided this also would be a good opportunity to further practice my python skills, particularly in object-oriented programming. I wrote a program that would simulate a basketball season in an imaginary league with 30 teams, each playing every other team twice.  It would store the box score for each game in a csv file, which I could then store in an SQL database and run queries against.  

Select the buttons below to reveal information for the components of the project you'd like to learn about. 

<div class="btn-group" role="group" aria-label="button group">
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#intro" aria-expanded="false" aria-controls="intro">
    Introduction
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#preprocessing" aria-expanded="false" aria-controls="preprocessing">
    Preprocessing
  </button>
  <button class="btn btn-outline-primary" type="button" data-bs-toggle="collapse" data-bs-target="#kmeans" aria-expanded="false" aria-controls="kmeans">
    K-means clustering
  </button>
</div>

<div class="collapse" id="intro">
    <div class="card card-body">
    </div>
</div>

<div class="collapse" id="preprocessing">
    <div class="card card-body">
    </div>
</div>

<div class="collapse" id="kmeans">
    <div class="card card-body">
        <h2>K-means Clustering Analysis</h2>
  <p>The objective of this analysis was to identify distinct customer segments within our marketing campaign dataset through K-means clustering.</p>
  
  <h3>Data Preprocessing</h3>
  <p>We started by preparing our data, which included removing duplicates, handling missing values, encoding categorical variables, and scaling numerical features to ensure a fair clustering process.</p>
  
  <h3>Determining the Number of Clusters</h3>
  <p>Using the elbow method, we determined the optimal number of clusters to be three, where the inertia's rate of decrease significantly slows down, indicating a balance between cluster cohesion and quantity.</p>
  
  <h3>Clustering and Visualization</h3>
  <p>After clustering the data, we visualized the clusters using a 2D scatter plot, reducing dimensions with PCA for a clearer understanding of our customer segments.</p>
  <img src="path_to_your_elbow_plot.png" alt="Elbow Plot" title="Elbow Method Plot">
  <img src="path_to_your_cluster_plot.png" alt="Cluster Plot" title="Cluster Distribution">
  
  <h3>Cluster Analysis</h3>
  <p>We analyzed each cluster's characteristics by examining centroids and feature distributions, identifying unique customer segments based on their behaviors and preferences.</p>
  
  <h3>Insights and Business Implications</h3>
  <p>Our analysis provided valuable insights into customer segmentation, allowing for targeted marketing strategies tailored to each segment's specific characteristics and needs.</p>
    </div>
</div>