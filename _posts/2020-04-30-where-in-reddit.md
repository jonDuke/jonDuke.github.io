---
layout: post
title: Group Project- Where in Reddit?
tags: [Document Similarity, Cross-functional]
comments: true
---

# Group Project: Where in Reddit?

It's been a while since I could add anything here, classes got busy.  But we just finished a capstone group project, and that's the perfect thing to put up here.

In this project, I helped build a web app that could help suggest where to put a text post on Reddit.  In case you aren't familiar with Reddit, it is a social site that consists of thousands of specialized message boards called "subreddits".  Since there are so many out there, it can be difficult sometimes to decide which subreddit to post in.

This was a cross-functional group project, we had 3 data scientists (including myself), and 4 web developers.  Additionally, this was only a one-week project so there are many places we could improve it given more time.  The experience communicating and working with people from other specialties is likely the most valuable thing I got here, but I will be focusing on the data science portion in this article.  To learn more about the rest of it, you can visit the site itself.  (It's very unpolished, one week was not enough for the web team to finish everything they wanted to.  But the post predictions do work.)

[https://whereinreddit.netlify.app/](https://whereinreddit.netlify.app/)

In short, the app allows the user to input a text post and then shows a couple subreddits that it might fit into.  This boiled down to a document classification problem, so we needed to collect existing Reddit posts to train the model with.

## Data Collection

Connecting to the Reddit API and pulling posts was relatively easy, but we did not have a good way to search for subreddits to actually consider.  Obviously we couldn't use all of them, there are over 100 thousand out there.  In the end we took a list of the most popular subreddits by subscriber count, added a few others that we hand-picked, and pulled posts.  

From each of them we took 1000 posts from the current "hot" postings, and another 1000 from the top of all time postings, then filtered and only kept the text posts.  This gave us a collection of just over 31 thousand documents, spread between 50 different subreddits.  Not a bad start!

Given more time, this is a key area I would focus on to further improve the project.  We ended up with anywhere between 100 to 2000 text posts per subreddit, so the sampling was a bit inconsistent.  There was no quick way to pull more posts so we moved forward with what we had, but getting more samples for the less represented subreddits would be good.  I'd also like to be more selective in the subreddits chosen, in order to get a good variety of topics and avoid overlap.

## Model Selection

Now that we had the data, the next step was picking a model.  As a team, we fit several different types of models and compared them all by accuracy, precision, and recall.  We tried all kinds of classifiers, such as Logistic Regression, Naive-Bayes, K Nearest Neighbors, and even some neural network implementations, but the clear winner was the good old Random Forest.  

Scikit-learn's default random forest classifier achieved a 66% test accuracy, especially impressive considering it had to predict between 50 classes.  The runner up was K Nearest Neighbors, with 53% accuracy.  We saw similar patterns with the precision and recall between each model.

The Random Forest also had 100% training accuracy at this point, so it was pretty obviously over-fitting, but it still had significantly higher test accuracy so we moved forward with it.

## Model Refinement

Diving deeper into things, my team-mate focused on applying NLP techniques while I looked into hyperparameter optimization.  In the end, we used techinques like stripping punctuation and removing stop words, but since I was not the one that went through that process I can't really describe it here.

For my part, I was a bit concerned about file size since this model needed to be deployed online.  The default Random Forest was 1GB in size after using Python's pickle process to save it, so I focused on reducing that while maintaining accuracy.  Two parameters worked best for this: 'n_estimators' and 'max_depth'.

Some exploration revealed that the individual trees were growing to a max depth of almost 1000.  I ran a grid search with cross-validation, and found I could reduce max_depth to 200 while keeping the same accuracy.  Reducing n_estimators did also reduce accuracy, but not by too much.  In the end, we used 10 estimators and ended up with a pickled model that was only about 100MB, much easier to deploy and still had 63% test accuracy.

## Integration

Since this was a group project, of course we needed to make this model usable by our web dev teammates.  Again, this was a short one-week project with students who have never worked together, so we kept things simple and deployed a web app using methods we were already familiar with.  The data science team simply deployed a web app on Heroku, which provided a public API link that the web team could connect to.

We took the text post that the user enters, then returned a JSON object with the 5 most likely subreddits that the model predicted.  The end result is a working web app.

Again, you can learn more about the project and everyone who helped with it over here: [https://whereinreddit.netlify.app/html/aboutus](https://whereinreddit.netlify.app/html/aboutus)
