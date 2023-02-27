---
layout: post
title: "Linear Regression: Intuition"
date: 2022-03-17
categories: [Machine Learning, Data Science]
cover: /assets/images/posts/2022-03-17-linear-regression/stock-cover.jpg
katex: true
---

In this post, we explore the intuition behind linear regression. While a relatively simple algorithm, it employs a bunch of ideas and techniques that are universal across machine learning.

> This post is part of my ongoing series, [The Intuition of Machine Learning](/intuition-of-ml).

Linear regression helps you find a pattern or trend in your data. It determines what attributes of your data influence the trend and by how much. Once we understand the trend of the data, we can make predictions based on it. Thus, we utilize historical data to make future predictions.

Let's take a simple and very popular example: _predicting house prices_. Our dataset is a bunch of pairs: the area of a house and its price. It tells us the price for which a house of a certain area was sold for.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page1.png" style="  width: 700px">
</center>

This is obviously naive and oversimplified, since the price of a house is not just dependent on its area but also on other factors such as its locality, the number of rooms, number of bathrooms, condition of the house, its age and dozens more. These are what we call **features** in machine learning. Features influence the _trend_.

For the sake of brevity, we are going to work with just one feature in this example i.e. the area in square feet. If we were to plot a graph of price vs. area, it would look something like this:

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page2.png" style="width: 700px">
</center>

Just by eyeballing this, we can clearly see the trend here: _The price increases as the area does._ It's a linear relationship. But that's not why we call this linear regression if you're curious. We'll worry about terminology later. All we care about is that the trend of the data would be a line like this:

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page3.png" style="width: 700px">
</center>

As humans, this comes quite intuitively to us. But why? Why do we think the trend of these data points is roughly along this line? Probably because most of the points pass through the trend line, or are close to it. That makes a lot of sense: if the trend needs to be representative of the underlying data, then it needs to be as close as possible to it. If that's the case, the absolutely perfect fit for this dataset would be a monstrosity like this:

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page4.png" style="width: 700px">
</center>

This curve passes through all the points. Your intuition probably says that this is excessive. This curve is trying _too hard_ and letting each and every point influence it _a bit too much_. This phenomenon is called **overfitting**. This is the [yes-man](https://www.merriam-webster.com/dictionary/yes-man){:target="\_blank"} of machine learning models. Or when you go around seeking career advice from a dozen different people and end up confused and overwhelmed because you don't know what you want in the first place. If those analogies don't make any sense to you right now, don't fret, we'll talk extensively about overfitting later.

We certainly don't want that grotesque monstrosity; the line we had previously seems like a good choice. It was not getting influenced too much by each point and just seemed more _balanced_.

Your next question would be, what do we do with this line? Well, we can make predictions! If we wish to determine the price of a new house, we can take its area, and make a projection on the trend line. From there, we make another projection onto the y-axis and voilá! Now you know the price of the house, predicted by our little machine learning model.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Projection.png" style="width: 700px">
</center>


Now if you think about it, there could be many such lines that roughly represent the trend of the dataset. All of the below seem to follow the _trend_, more or less. There could also be so many more. How do we pick the best one?

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page5.png" style="width: 700px">
</center>

> But wait, what does **best** even mean?

Let's try to quantify that a bit. We previously said that the trend line needs to be as close as possible to all the data points. We could quantify this by calculating the distance from each point to the line and adding up these distances. The line with the lowest sum of distances would be _the best_.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page6.png" style="width: 700px">
</center>

So, the overall algorithm would be as follows:

```py
for line in trend_lines:
    sum = 0
    for point in dataset:
        sum += distance_between(point, line)

    if sum is least so far:
        best_line = line
```

Now onto a tiny but important detail: If we simply calculate the distance between the points and the line, it could be positive or negative depending on whether it's above or below the line. We don't really care about the sign difference. We only care about how close the line is to the points i.e. we just need the magnitude of the distance. Your immediate response would be to then take the absolute value of the distance. While that is valid, the most common technique is to rather **square** the distances. There are a couple of reasons behind this choice but the most important one is that squares massively simplify the math later on.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page7.png" style="width: 700px">
</center>

Thus, our algorithm changes slightly to:

```py
    sum += distance_between(point, line) ^ 2
```

This means that the output of linear regression is but a humble **line**. More generally speaking, it is a **curve**. If you recall from middle school, a line has the following equation:

$$y = mx + c$$

Here, $m$ is the slope of the line, and $c$ is the y-intercept aka the point on the y-axis where the line intersects. $x$ and $y$ are variables. Thus, $m$ and $c$ characterize the line. When we say that we take the least sum of squared distances for a few trend lines, we are tweaking $m$ and $c$ slightly and finding the ones that offer the best fit. The takeaway here is that the final output of linear regression is going to be coefficients of a line: $m$ and $c$.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page8.png" style="width: 700px">
</center>

In our housing prices example, if the area of the house is zero, the price would also be zero. Thus, our trend line would pass through the origin and the $c$ term would be eliminated. The equation simplifies to:

$$ y = mx $$

<!-- Maybe add trend line image here -->

If $y$ is the price and $x$ is the area, then $m$ is the influence of the area on the price of the house. It is the _weight_ that we give to the area. If a house has an area of $x=1000$ sqft., and we weigh the area by $m = 100$, the price of the house would be $100K.

If the line was **not** passing through the origin, however, we return to the original equation of the line i.e. $ y = mx + c$.

<center>
<img src="/assets/images/posts/2022-03-17-linear-regression/Page9.png" style="width: 700px">
</center>

$c$ is not a weight, since it does not influence any feature. Look at the equation, $c$ sits there by itself, not being multiplied by any variable. It simply shifts the line towards the data, in this case above. In machine learning lingo, $c$ is the **bias** term.

Thus, in the end, linear regression is just a process of determining the best **weights and biases**, as are many things in machine learning. Using these, we can then make predictions. We can also easily extend this idea to use multiple features, each of which gets its own weight. That's called multi-variate linear regression, but that's for another time.

In the next post, I formalize the ideas discussed in this post and derive the normal/closed-form equation to find the most optimal weights and biases.
