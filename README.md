---
author:
- Andry Rafaralahy
bibliography:
- bibli.bib
title: WiFi Fingerprinting
---

# Introduction

The given task was to build and train a WiFi based localization system
using data from the UJIIndoorLoc dataset.

The data set is already split into train/validation sets. The provided
validation set was used as a test set. The training dataset was split
into training and validation sets. The split was done based on USERID
column so that training set contains 70% of the data and the validation
set contains 30%. The resulting validation set was used to tune the
hyperparameters of each considered model.

# k-Nearest Neighbors

A first simple approach to estimate position from a vector of RSSI
values is to use a k-Nearest Neighbors algorithm. We can estimate floor
and building by majority vote and longitude and latitude by
interpolation.

The contribution of each neighbor can be weighted with a function of its
distance to the considered vector. This approach is similar to the WKNN
approach described in [@moreira].

::: {#tab:knn_params}
              k   Weighting function
  ----------- --- --------------------
  Building    5   $1/d^2$
  Floor       5   $1/d^2$
  Longitude   3   $1/d^2$
  Latitude    3   $1/d^2$

  : Chosen hyperparameters values for the kNN models
:::

# Deep learning based methods

### Longitude and latitude estimation

Longitude and latitude are jointly estimated using a multilayer
perceptron.

![Overview of the architecture used for longitude and latitude
estimation](mlpreg.png){#fig:mlp_reg width="50%"}

Batch normalization was applied to each layer. White Gaussian noise was
added to inputs in order to reduce overfitting. Based on the results of
experiments done on the validation set, standard deviation of 0.1 was
chosen.

Training was done with an initial learning rate of 0.001. The \"reduce
on plateau\" strategy which consists in reducing the learning rate when
validation loss stops decreasing was used. The model was trained for 200
epochs.

::: {#tab:mlp_reg_params}
                           Initial learning rate   Learning rate decay   Batch size   Input noise $\sigma$
  ------------------------ ----------------------- --------------------- ------------ ----------------------
  Hyperparameters values   0.001                   0.1                   128          0.1

  : Chosen hyperparameters values for longitude/latitude MLP regressor
:::

### Building and floor classification

Building and floor classification is done with a simpler architecture,
with only one hidden layer.

![Overview of the architecture used for building and floor
estimation](mlpclf.png){#fig:mlp_clf width="20%"}

Training was done with an initial learning rate of 0.0001.

::: {#tab:mlp_clf_params}
                           Initial learning rate   Learning rate decay   Batch size   Input noise $\sigma$
  ------------------------ ----------------------- --------------------- ------------ ----------------------
  Hyperparameters values   0.0001                  0.1                   128          0.1

  : Chosen hyperparameters values for building/floor MLP classifier
:::

# Results

### Longitude and latitude estimation

To evaluate the accuracy of our models on longitude/latitude estimation
the mean position error (ME) and root-mean-square error were considered.

         Validation ME   Validation RMSE   Test ME    Test RMSE
  ------ --------------- ----------------- ---------- -----------
  k-NN   **8.96**        15.68             **9.26**   16.00
  MLP    9.86            **14.14**         9.74       **13.66**

  : Results for the longitude/latitude estimation task

### Building and floor estimation

::: {#tab:build_floor_res}
         Validation building accuracy   Validation floor accuracy   Test building accuracy   Test floor accuracy
  ------ ------------------------------ --------------------------- ------------------------ ---------------------
  k-NN   0.9732                         0.9072                      0.9953                   0.8920
  MLP    **0.9984**                     **0.9282**                  **0.9990**               **0.9298**

  : Results for the building and floor estimation tasks
:::

# Conclusion

The k-NN approach gave the best results on longitude and latitude
estimation. However it was outperformed by the neural network based
approaches on building and floor estimation.

For some reason, the results for the k-NN approach on building and floor
classification do not match [@moreira] WKNN results. More thorough
hyperparameter tuning is probably needed to achieve the same accuracy.
