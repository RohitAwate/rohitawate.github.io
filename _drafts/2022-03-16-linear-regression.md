---
layout: post
title: Linear Regression
date: 2022-03-16 17:45:00
categories: Machine Learning, Data Science
summary: About algo
cover: /assets/images/posts/2022-03-16-linear-regression/stock-cover.jpg
katex: true
---

In this post, we explore the intuition behind linear regression. While a relatively simple algorithm, we shall be learning a bunch of ideas and techniques that are used universal across machine learning.

Linear regression helps you find a pattern or trend in your data. It determines what attributes of your data influence the trend and by how much. Once we understand the trend of the data, we can make predictions based on it. Thus, we utilize historical data to make future predictions.

Let's take the simplest and most popular example: predicting house prices. Your data is a bunch of pairs: the area of a house and its price. It tells you for what price a house of a certain area sold for. This is obviously naive and an oversimplification, since the price of a house is not just dependent on its area but also on other factors such as the locality of the house, the number of rooms, number of bathrooms, condition of the house, the year it was built and dozens more. These are what we call **features** in machine learning. Features influence the _trend_.

For the sake of brevity, we are going to work with just one feature in this example i.e. the area in square feet. If we were to plot a graph with the area on the X-axis and the price on the Y-axis, we would see a graph that looked something like this.

# TODO: Graph of data

Just by eyeballing this, we can clearly see the trend here: _The price increases as the area does._ It's a linear relationship. But that's not why we call this linear regression, if you're curious. We'll worry about terminology later. The point I am trying to make here is that the trend of the data would be a line like this:

# TODO: Graph with trend line

As humans, this comes quite intuitively to us. But why? Why do we think the trend of these data points is roughly along this line? One possible explanation would be that most of the points pass through the line, or are close to it. That makes a lot of sense: if the trend needs to be representative of the underlying data, then it needs to be as close as possible to it. If that's the case, the absolutely perfect fit for this data would be a monstrosity like this:

# TODO: Graph passing through all points

This curve passes through all the points. My intuition says that this is excessive. This curve is trying _too hard_ to please each and every point in the dataset and letting it influence it _a bit too much_. This phenomenon is what we call **overfitting**. This is the [yes-man](https://www.merriam-webster.com/dictionary/yes-man) of machine learning models. If that doesn't make any sense to you right now, don't fret, we'll get to it later.

We certainly don't want that grotesque monstrosity and the kind we had before is the one we are looking for. But if you think about it, there could be many lines which roughly represent the trend.

# TODO: All possible trend lines

All of the above seem to follow the _trend_, more or less. There could also be so many more. How do we pick the best one? But before that, what does best even mean? Let's try to quantify that a bit. We previously said that the trend line needs to be as close as possible to all the data points. We could quantify this by calculating the distance from each point to the trend and adding up these distances. The trend line with the lowest sum of distances would be _the best_.

# TODO: One trend line and show distance of points from each point and sum them up

So, the overall algorithm would be as follows:
```py
for line in trend_lines:
    sum = 0
    for point in data:
        d = distance_between(point, line)
        sum += d

best_line = line with minimum sum
```

Now onto a tiny but important detail. If we simply calculate the distance between the points and the line, it would be positive for a point above the line and negative for one below the line. We don't really care about the sign difference. We just need the magnitude of the distance. Your immediate response would be to take the absolute value of the distance, then. While that is valid, the most common technique is to rather **square** the distances. The reason why we do that is to simplify the math later on. 

# TODO: Zoomed in, Line passing through middle, a point above and down

This means that the output of linear regression is but a humble **line**. More generally speaking, it is a **curve**. If you recall from middle school, a line has the following equation:

$$y = mx + c$$

Here, $m$ is the slope of the line, and $c$ is the y-intercept. $x$ and $y$ are variables. Thus, $m$ and $c$ characterize the line. When we say that we take the least sum of squared distances for a few trend lines, we are tweaking the $m$ and $c$ slightly and finding the one that fits best. Thus, my point here is that the final output of linear regression is going to be these coefficients of a line: $m$ and $c$.

# TODO: y = m1x + c1 and so on upto n and then their corresponding costs and then pick min