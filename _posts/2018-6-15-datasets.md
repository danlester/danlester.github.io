---
layout: post
title: Tensorflow new Datasets API
---

I was trying to get Tensorflow 1.8 to run under [Google Cloud ML](https://cloud.google.com/ml-engine/) - potentially to be distributed, but more importantly just to use a faster processor than my laptop. A complication is that I needed my input data to be stored on Google Cloud Storage. For a single machine, it would probably be fine just to make sure I load the file directly into memory from Cloud Storage, but especially for use in distributed settings, Tensorflow 1.8 ideally sees you use their new [Datasets API](https://www.tensorflow.org/programmers_guide/datasets) to load the data. This gets quite complicated, especially if you need to do processing on the raw input CSV data.

This describes my example of doing 'post-processing' of a CSV column.

I was looking at the Kaggle competition [Facial Keypoints Detection](https://www.kaggle.com/c/facial-keypoints-detection), and in particular trying to translate [Daniel Nouri's suggestions](http://danielnouri.org/notes/2014/12/17/using-convolutional-neural-nets-to-detect-facial-keypoints-tutorial/) which are based on various frameworks, into a raw Tensorflow 1.8 implementation.

The raw CSV input files have 31 columns - 30 floating point numbers followed by an 'Image' string which contains 96*96 space-separated integers describing a greyscale image. For example:

66.03356390,39.002273684,30.2270075188, ... ,43.13070679,84.48577441,238 236 237 238 240 240 239 241 241 243 240 239 231 212 190 173 148 ... 104 92 79 73 74 73 73 74 81 74 60 64 75

The first 30 columns are easy enough, and the whole thing is simple using pandas (see Daniel Nouri's blog post). But the Datasets API requires us to write a native Tensorflow graph to do the processing. (OK an alternative would be just to translate the data file ahead of time into a native format.) Here is a solution, missing the scaling that Daniel has:

{% gist f7cdbc741cf06317ff3870c52bb3474a %}

The key Tensorflow components are: tf.data.TextLineDataset, tf.decode_csv, tf.string_split, tf.sparse_to_dense, tf.string_to_number.
