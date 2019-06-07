---
layout: post
title: Introducing Jupyter Innotater
---

In a data science or machine learning project, you may prepare and study images or other data within a Jupyter notebook then need to annotate the data to augment the training or fix errors in your source data.

My new Jupyter notebook widget, [Jupyter Innotater](https://github.com/ideonate/jupyter-innotater), allows you to annotate data including image bounding boxes inline within your Jupyter notebook in Python. Its flexible API allows easy selection of interactive controls to suit your datasets exactly.

Since you are already working within a Jupyter notebook, the Innotater works inline allowing you to interact with your data and annotate it quickly and easily, syncing straight back to your input data arrays or matrices.

Within Jupyter, you can easily home in on problem input data - perhaps only misclassified images - so you can step through and adjust bounding boxes just for those items.

The Innotater widget is designed with a flexible API making it quick and easy to get started exploring your dataset, guessing how to work with your data without explicit configuration where possible.

The project is currently in ALPHA development phase, and I appreciate all feedback on any problems including details on how the current code works or fails to work for the structure of your particular projects.

Find out [more on GitHub](https://github.com/ideonate/jupyter-innotater) or read the [full documentation here](https://jupyter-innotater.readthedocs.io/). I've also written an [article on Towards Data Science](https://towardsdatascience.com/clean-up-your-own-model-data-without-leaving-jupyter-bdbcc9001734) describing a full end-to-end project using the Innotater for drawing bounding boxes to improve model predictions.


