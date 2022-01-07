---
layout: post
title:  "Law Area Classification using Text Mining"
date:   2019-10-21 15:03:36 +0530
categories: python
excerpt: "Categorizing court documents using NLP"
image: "/images/textclassification.jpg"
---


> **_NOTE:_** Part of MSc. Computing Science curriculum.
> 
> Find a download link for the PDF report at the bottom of the page.



Categorizing text documents into predefined categories can be an exhaustive manual task. In this post, we try to automatically classify law documents from *rechtspraak.nl* into their corresponding categories. We will get familiar with text mining using machine learning, and reflect on the results.
Keywords: *Law Classification, Text Classification, Bag-of-Words, TF-IDF, Linear SVM, One-Versus-Rest, Text Weighting, Supervised Learning*

## Abstract
Text classification is not new; there are several studies showing that Support Vector Machines (SVMs) with TF-IDF and Naive Bayes approaches do a decent job in text classification. By looking at parts of the law documents, we try to improve classification scores. Because we know that introductory text of dutch law documents of Rechtspraak.nl contain valuable information that is relevant for classification, we investigate whether applying extra (heavier) weighting to this introductory text will improve classification performance. Each document can have multiple law categories attached. Because SVMs cannot assign multiple labels (multi-label classification) out of the box, a One-Versus-Rest (OVR) scheme is used. This also solves the problem with overlapping categories. This approach seems to improve performance (F1-score) significantly, but with a very small percentage (0.22%).


## Introduction
With increasingly growing court cases and different resources to grab information from, it is a valuable addition to have these resources automatically classified in their corresponding categories. For this project we got assigned a classification problem of law documents from Legal Intelligence. Legal Intelligence wants to classify law documents into categories by classifying on textual data. The data that is being used is public property of the dutch Court (Rechtspraak.nl), and consists of lawsuits and their verdicts. Each document has been assigned one or more law category (labels), and it is our task to assign these documents their corresponding labels. Although the documents consists of XML data with multiple information fields, we should solely use the document text as our information source. We will go into the details of how we can improve the text classification by amplifying parts of the document we think that are relevant.

## Research Question
After analyzing some documents, it becomes clear that nearly every document starts with crucial information that could determine the law area of that particular document. Information regarding the location of the sitting, which tribunal it addresses, but sometimes also small summaries of previous sittings seems to be listed at the introductory text of each document. It would be interesting to see whether this information contributes more to classification than the rest of the text. Bruninghaus [2] already suggested to focus on smaller sentences of the document instead of classifying complete documents, which will add knowledge during parsing of documents. Our research question is thus:

*Would heavier weighting on introduction text improve the classification score of an SVM algorithm on dutch law category classification?*


## Methods
We choose to tackle this problem with a bag-of-words unigram approach. This means that every word in the text is considered to be a feature in the classifier. Bigram approaches were also considered, but due to limitations in computing resources this could not be tested (note: a bug between Joblib & Pickle made multiprocessing on computing clusters with large datasets impossible). 

Research regarding classification of law documents have already been done with Naive Bayes and C4.5 approaches [9]. However, SVM seems to outperform Naive Bayes and Neural Networks (perceptron and three-layers neural networks including a hidden layer) on category classification [4]. Further investigation of neural networks
found that these neural networks could be superior towards SVM approaches [7]. We still stick to SVM because of its simplicity and scalability.

The dataset contains multi-class labels, where each document can be assigned to multiple labels. Because standard SVM approaches are not multi-class, and are not able to assign multiple labels, we choose the One-Versus-Rest (OVR) approach. This means that for each class that can be assigned, there is a need for a separate SVM classifier. Later on we will discuss this pipeline in depth.


# Pipeline
1. Preprocessing
2. CountVectorizer
3. TF-IDF Transformer
4. Support Vector Machine (SVM)
5. One-VS-Rest Classifier

### Preprocessing
The dataset that was given consists of 1.8GB XML files (around 290.000 documents). Each file corresponds to one entry in the rechtspraak.nl database. The XML files contain a lot of extra metadata. Because we want to classify on just textual data, this other metadata is excluded. The metadata parameters we are interested in are the identifiers *Uitspraak* and *Rechtsgebied*. These identifiers correspond to our data and labels respectively.

### CountVectorizer
The second step in the pipeline is the Scikit's *CountVectorizer*. This function converts documents into matrix representations. Each vector entry in the matrix is determined by splitting a document into words. The documents are splitted by the *CountVectorizer* by selecting strings that have two or more alphanumeric characters, which means that special characters are being ignored. We also make sure that all words are transformed into lowercase format. This matrix forms the foundation of our bag-of-words model.

### TF-IDF Transformer
The third step in the pipeline is the TfidfTransformer. Using term frequency-inverse document frequency (TF-IDF) with a bag-ofword approach is a widely used practice in NLP [11]. TF-IDF is a way to weight text by taking into account the word distribution in the corpus. A word gets a higher TF-IDF value when a word appears often in a document (term frequency), but less often in the document corpus (inverse document frequency). Words that are occurring often in both documents and the document corpus (ex: stopping words) gets assigned less weight. The advantage of this is that less informative words get less weight, whereas less occurring informative words gets boosted weights. The TfidfTransformer takes a word frequency matrix as input and will calculate its TF-IDF representation using weights. Default L2 normalization is applied to normalize word vectors, and Laplace smoothing is applied to prevent ’dividing by zero’ errors. Other approaches that include the usage of the newer BM25 instead of TF-IDF have been considered. However, [6] concludes that term frequency transformations such as BM25 show no remarkable effect on classification, and that an SVM classifier seemed ’state of the art in this domain’.

### SVM & One-Versus-Rest (OVR)
The classifier where we test our custom weighting with is the Support Vector Machine (SVM). Because our dataset is so large, we take an SVM that easily scales with the amount of data. As stated in [4], linear SVMs are fast in both training and evaluation, and are accurate text classifiers. Also, linear SVMs scale better to large amounts of data. Because SVMs are binary classifiers, we utilize a One-Versus- Rest classifier scheme to perform multi-label classification (also called the binary relevance method). This means that for each class, a separate SVM is produced to solve that particular class. In our dataset this means the scaffolding of 29 SVMs through an OVR scheme. Another reason to utilize a OVR scheme is because there can be overlapping between categories. When using only one SVM, the decision boundaries can be influenced by this overlapping of categories, which results in lower classification scores. SVM supports the same properties for classifying text as our bag-of-words approach. This approach also has a high dimensional feature space, few irrelevant features and sparse instance vectors (most documents have a vector that only a few entries that are not zero) [5]. Also, "SVMs eliminate the need for feature selection, making the application of text categorization considerably easier". The need for feature selection does not apply to SVM text classification techniques. It has been investigated that a classifier which only uses words that are the least to contribute to the classification performance, still have a better performance than when performing a random action (chance level) [5]. 

## Results
For each experiment that is run, 300 stratified crossvalidations are performed against each dataset, with a test size of 30%. For evaluation of scores, an F1-weighted measure is used. F1-weighted calculates the metrics for each label, and averages this by considering the amount of support per class. Classes with lower support have less influence on the averaged F1-score. The F1-score of the baseline experiment is 0.96055, where the F1-score of the experimental condition is 0.96283. The increase of the experimental condition in performance is minimal (\~0.22% increase), but because we have 300 crossvalidations to support these results, this difference can be big enough to be significant. Table 2 shows us the individual class results for the experimental condition. Classes with less than 20 supporting queries are left out in the table for ease of visualization. Most of these left out classes have an F1-score of 0.00, because there is too less training data available for those classes. It is observed that classes that do have many training samples and thus high support generally have a better performances.. 

## Conclusion
Weighting introduction texts heavier on dutch law documents show significantly better results in comparison to the normal weights (baseline). However, these mean differences are very small (\~0.22% increase in performance, to an F1-score of 0.9628). There is much more benefit to be gained from other measures like Part of Speech tagging or semantic analysis. Upon analysis of results, we found out that some classes are superclasses of other classes. It was found that the class label Bestuursrecht is a superclass of all the classes Socialezekerheidsrecht, Vreemdelingenrecht, Belastingrecht, Omgevingsrecht, Bestuursprocesrecht, Ambtenarenrecht, Bestuursstrafrecht and Europees bestuursrecht. In Table 3, we can see that the frequency of every law area corresponds with the frequencies in Table 1, which suggests that Bestuursrecht is a superclass. Because we did not have any knowledge about law areas, we assumed that each label was an independent law area with no hierarchy. However, because of multi-label classification there exists a separate classifier for each class, so overlap between super- and subclasses do not cause interference in performances. Another point that should be made is that we do not check whether each document gets assigned all of their labels. Instead performance is calculated whether an individual class belongs to a document correctly. 

## Download Paper

The latex report can be download as pdf below:

**[Custom Text Weighting on Law Area Classification](../../files/Custom_Text_Weighting_on_Law_Area_Classification-Ruben-Kluge.pdf)**


