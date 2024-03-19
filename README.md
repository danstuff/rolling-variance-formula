## An Efficient Rolling Variance Calculation
Variance is a very useful statistic when trying to analyze a data set. Traditional methods of calculating variance rely on access to the entire data set at once, which can consume a significant amount of memory if your data set is large. However, it is possible to perform a “rolling” variance calculation, in which the variance is calculated using only a few variables and continually updated as new data comes in. The benefit of this is two-fold:
 - Avoids having to keep a large record in memory of every data point required for the variance calculation.
 - Distributes the resource cost of the variance calculation to reduce CPU load.

Since rolling variance is not very well documented online, I thought it would be useful to have a page which describes how I came about this particular method. All equations are written in pseudocode to make them more programmer-friendly.

For starters, let’s look at the tried-and-true average and variance formulae:

```
average = sum(x) / length;
variance = sum((x - average)^2) / length;
```

Here, `x` is a placeholder for any datapoint in the set, so `sum(x)` is the sum of all datapoints.

To make our rolling calculation simpler, we can work on the sum of squares instead of variance. The sum of squares is just the variance multiplied by the length of the data set, so it will be trivial to convert back to variance later.

```
square_sum = sum((x - average)^2);
```

A rolling average calculation is pretty simple to derive. We can just multiply the average by the old length to get the sum, add the new datapoint to the sum, then divide by the new length:

```
new_average = ((old_average * old_length) + x) / new_length;
```

To derive a rolling variance calculation, we will have to find a way to add new data points after the `square_sum` has been calculated. The easiest way to do that is to find the difference between `new_square_sum` and `old_square_sum`. 

It may be tempting to do something like this:

```
new_square_sum = old_square_sum + (x - average)^2;
```

This is partially correct. However, the previous values from `old_square_sum` need to be adjusted to reflect the fact that the average changes with each new value `x`. So we need to have two average values, `new_average` and `old_average`.

Let’s start by writing two separate equations for `new_square_sum` and `old_square_sum`. Let’s also introduce `delta_average`. This will be the difference between `new_average` and `old_average`, and will help us to further simplify. With this, we should be able to get the difference between `new_square_sum` and `old_square_sum` to find our rolling variance formula.

```
old_square_sum = sum((x - old_average)^2);
new_square_sum = sum((x - (old_average+delta_average))^2);
```

We can calculate the square in each equation like so:

```
old_square_sum = sum(x^2 - 2*x*old_average + old_average^2);
new_square_sum = sum(x^2 - 2*x*(old_average+delta_average) + (old_average+delta_average)^2);
               = sum(x^2 - 2*x*old_average - 2*x*delta_average + old_average^2 + 2*delta_average*old_average + delta_average^2);
```

Next, we take the difference between these two equations to find `delta_square_sum`. This is what we’ll have to add to `old_square_sum` to get `new_square_sum`.

```
      sum(x^2 - 2*x*old_average - 2*x*delta_average + old_average^2 + 2*delta_average*old_average + delta_average^2)
   -  sum(x^2 - 2*x*old_average + old_average^2)
________________________________________________
      sum(-2*x*delta_average + 2*delta_average*old_average + delta_average^2)
```

Next we can factor everything but `x` out of the sum by multiplying it by `old_length`. 

```
delta_square_sum = sum(-2*x*delta_average + 2*delta_average*old_average + delta_average^2);
                 = -2*delta_average*sum(x) + 2*delta_average*old_average*old_length + delta_average^2*old_length;
```

Because we are multiplying by `old_length`, we will have to account for the new variance datapoint by adding it back in later.

Since we are not yet accounting for the new datapoint, `sum(x) = old_average * old_length`, so we can substitute that in as well:

```
delta_square_sum = -2*delta_average*old_average*old_length + 2*delta_average*old_average*old_length + delta_average^2*old_length;
                 = delta_average^2*old_length;
                 = (new_average - old_average)^2 * old_length;
```

Now that we have a simplified `delta_square_sum`, we can add it to `old_square_sum` plus our new variance datapoint `(x - new_average)^2` to get `new_square_sum`.

Thus, a rolling sum of squares calculation can be performed like so:

```
new_square_sum = old_square_sum + old_length*(new_average - old_average)^2 + (x - new_average)^2;
```
