---
title: ""
permalink: word #unique word that will be used as url basesite.com/[word]
header:
  teaser: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
  #overlay_image: assets/images/[thing]
  overlay_filter: 0.3
  show_overlay_excerpt: false
categories:
#  - Guides#  - Guides

#  - Portfolio
#  - Uncategorized
tags:
  # - PowerBI
  # - Visualization
#toc: true
published: false

---





Key Words
Attribute: a data type (e.g. "Mileage")
Feature: a data type plus its value (e.g. "Mileage = 15,000")


Cost Function: Measures how bad your algorithm is by measuring distance between the predictions and the training data

Types of Learning
Supervised Learning: the training data already has the labels that we want it to fit on to new data (e.g. identify if mail is spam, using training data containing mail already identified as spam)




Unsupervised Learning: dynamically identify new labels/groupings in a set of unlabeled data.
  Clustering: identify groupings of similar data. (e.g. common types of visitors to a park)
  Visualization: given a lof of compex data, the system tries to output a 2d/3d representation of that data. A person may then be able to use that to glean their own realizations about it (e.g. given attributes about common objects [truck, chair, human], map that data by similarity to each other node. What shows up close/far away from eachother?)
  Dimensionality reduction / Feature Extraction: identify and merge corrolated features into one to reduce complexity. (e.g. if a car's mileage is highly correlated with age, this could be combined into a new feature.) This can be done before feeding into another algorithm to improve speed.
  Anomoly Detection: Look for outliers in data.
  Association Rule Learning: Look for relationships between attributes (e.g. If a customer buys hot dogs, they have a high chance of also buying buns)

Semisupervised Learning: deal with labeled data that contains unlabeled dimensions. (E.g. a photo contains labeled data [date, size] and unlabled data [faces, objects])
Reinforcement Learning: a system (agent) observes an environment, performs actions, and get rewards/punishment in return. It then learns based off of this output what the best strategy is (policy)





Steps in an ML project
1. Look at the big picture
2. Get the data
3. Read through & visualize the data to gain initial insights
4. Prep the data for the algorithm (ETL)
5. Select a model (or models) and train it/them.
6. Fine tune
7. Present the model to stakeholders
8. Launch/Monitor/Maintain






<small>desc.</small>

<hr>


this is a <a class="thumbnail">hoverlink<span><img src="{{site.url}}{{site.baseurl}}/assets/reactionimages/mindblown.gif"><br></span></a>

and this is a [normal link](https://google.com)


and this is an image
![Image Name]({{site.url}}{{site.baseurl}}/assets/images/picfix_welcome.png)


> This is a quote
<br>
<small><cite>
> from a guy
</cite></small>

## title

### sub title



```
code block
```

Two image figure

<figure class="half">

<img src="../assets/images/Annotation 2019-03-12 100327.png">
<img src="../assets/images/Annotation 2019-03-12 100438.png">
<figcaption>caption </figcaption>
</figure>


Colons can be used to align columns.

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

There must be at least 3 dashes separating each header cell.
The outer pipes (|) are optional, and you don't need to make the
raw Markdown line up prettily. You can also use inline Markdown.

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3
