---
layout: post
title:  "Intermediate Ggplot2"
categories: R
tags:  ggplot
author: Nicol Huaraca
---
* content
{:toc}

Guide and exercises for ggplot2. Based in Datacamp.



# Intermediate ggplot2

[TOC]



## Stats with geoms

### Smoothing

```
# Amend the plot to add a smooth layer
ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point() +
  geom_smooth()
```

`geom_smooth()` using method = 'loess' and formula 'y ~ x'. Use the default method, which uses the LOESS model to fit the curve.

```
# Amend the plot. Use lin. reg. smoothing; turn off std err ribbon
ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE)
```

### Grouping variables

We'll continue with the previous exercise by considering the situation of looking at sub-groups in our dataset. For this we'll encounter the invisible `group` aesthetic.

`mtcars` has been given an extra column, `fcyl`, that is the `cyl` column converted to a proper factor variable.

```
# Amend the plot to add another smooth layer with dummy grouping
ggplot(mtcars, aes(x = wt, y = mpg, color = fcyl)) +
  geom_point() +
  stat_smooth(method = "lm", se = FALSE)
```

```
# Amend the plot to add another smooth layer with dummy grouping
ggplot(mtcars, aes(x = wt, y = mpg, color = fcyl)) +
  geom_point() +
  stat_smooth(method = "lm", se = FALSE) +
  stat_smooth(aes(group = 1), method = "lm", se = FALSE)
```

### Modifying stat_smooth

In the previous exercise we used `se = FALSE` in `stat_smooth()` to remove the 95% Confidence Interval. Here we'll consider another argument, `span`, used in LOESS smoothing, and we'll take a look at a nice scenario of properly mapping different models.

```
# Amend the plot to color by fcyl
ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point() +
  # Add a smooth LOESS stat, no ribbon
  stat_smooth(se = FALSE) +
  # Add a smooth lin. reg. stat, no ribbon
  stat_smooth(method = "lm")
```

```
# Amend the plot
ggplot(mtcars, aes(x = wt, y = mpg, color = fcyl)) +
  geom_point() +
  # Map color to dummy variable "All"
  stat_smooth(aes(color = "All"), se = FALSE) +
  stat_smooth(method = "lm", se = FALSE)
```

### Modifying stat_smooth (2)

In this exercise we'll take a look at the standard error ribbons, which show the 95% confidence interval of smoothing models. `ggplot2` and the `Vocab` data frame are already loaded for you.

`Vocab` has been given an extra column, `year_group`, splitting the dates into before and after 1995.

```
# Using Vocab, plot vocabulary vs. education, colored by year group
ggplot(Vocab, aes(education, vocabulary, color = year_group)) +
  # Add jittered points with transparency 0.25
  geom_jitter(alpha = 0.25) +
  # Add a smooth lin. reg. line (with ribbon)
  stat_smooth(method = "lm")
```

```
# Amend the plot
ggplot(Vocab, aes(x = education, y = vocabulary, color = year_group)) +
  geom_jitter(alpha = 0.25) +
  # Map the fill color to year_group, set the line size to 2
  stat_smooth(aes(fill = year_group),method = "lm", size = 2)
```

## Stats : sum and quantile

### Quantiles

Here, we'll continue with the `Vocab` dataset and use [`stat_quantile()`](http://www.rdocumentation.org/packages/ggplot2/functions/geom_quantile) to apply a quantile regression.

Linear regression predicts the mean response from the explanatory variables, quantile regression predicts a quantile response (e.g. the median) from the explanatory variables. Specific quantiles can be specified with the `quantiles` argument.

Specifying many quantiles *and* color your models according to year can make plots too busy. We'll explore ways of dealing with this in the next chapter.

```R
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  geom_jitter(alpha = 0.25) +
  # Add a quantile stat, at 0.05, 0.5, and 0.95
  stat_quantile(quantiles = c(0.05, 0.5, 0.95))
```

```
# Amend the plot to color by year_group
ggplot(Vocab, aes(x = education, y = vocabulary, color = year_group)) +
  geom_jitter(alpha = 0.25) +
  stat_quantile(quantiles = c(0.05, 0.5, 0.95))
```

### Using stat_sum

In the `Vocab` dataset, `education` and `vocabulary` are integer variables. In the first course, you saw that this is one of the four causes of overplotting. You'd get a single point at each intersection between the two variables.

One solution, shown in the step 1, is jittering with transparency. Another solution is to use [`stat_sum()`](http://www.rdocumentation.org/packages/ggplot2/functions/geom_count), which calculates the total number of overlapping observations and maps that onto the `size` aesthetic.

`stat_sum()` allows a special variable, `..prop..`, to show the *proportion* of values within the dataset.

```
# Run this, look at the plot, then update it
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  # Replace this with a sum stat
  geom_jitter(alpha = 0.25)
```

```
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  stat_sum() +
  # Add a size scale, from 1 to 10
  scale_size(range = c(1,10))
```

```R
# Amend the stat to use proportion sizes
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  stat_sum(aes(size = ..prop..))
```

```
# Amend the plot to group by education
ggplot(Vocab, aes(x = education, y = vocabulary, group = education)) +
  stat_sum(aes(size = ..prop..))
```

## Stats outside geoms

### Preparations

In the following exercises, we'll aim to make the plot shown in the viewer. Here, we'll establish our positions and base layer of the plot.

Establishing these items as independent objects will allow us to recycle them easily in many layers, or plots.

- [`position_jitter()`](http://www.rdocumentation.org/packages/ggplot2/functions/position_jitter) adds *jittering* (e.g. for points).
- [`position_dodge()`](http://www.rdocumentation.org/packages/ggplot2/functions/position_dodge) *dodges* geoms, (e.g. bar, col, boxplot, violin, errorbar, pointrange).
- [`position_jitterdodge()`](http://www.rdocumentation.org/packages/ggplot2/functions/position_jitterdodge) *jitters* **and** *dodges* geoms, (e.g. points).

As before, we'll use `mtcars`, where `fcyl` and `fam` are proper factor variables of the original `cyl` and `am` variables.

```R
# From previous step
posn_j <- position_jitter(width = 0.2)
posn_d <- position_dodge(width = 0.1)
posn_jd <- position_jitterdodge(jitter.width = 0.2, dodge.width = 0.1)

# Create the plot base: wt vs. fcyl, colored by fam
p_wt_vs_fcyl_by_fam <- ggplot(mtcars, aes(fcyl, wt, color = fam))

# Add a point layer
p_wt_vs_fcyl_by_fam +
  geom_point()
```

### Using position objects

Now that the position objects have been created, you can apply them to the base plot to see their effects. You do this by adding a point geom and setting the `position` argument to the position object.

The variables from the last exercise, `posn_j`, `posn_d`, `posn_jd`, and `p_wt_vs_fcyl_by_fam` are available in your workspace.

```
# Add jittering only
p_wt_vs_fcyl_by_fam +
  geom_point(position = posn_j)
```

```
# Add jittering and dodging
p_wt_vs_fcyl_by_fam +
  geom_point(position = posn_jd)
```

### Plotting variations

The preparation is done; now let's explore [`stat_summary()`](http://www.rdocumentation.org/packages/ggplot2/functions/stat_summary).

*Summary statistics* refers to a combination of *location* (mean or median) and *spread* (standard deviation or confidence interval).

These metrics are calculated in `stat_summary()` by passing a function to the `fun.data` argument. `mean_sdl()`, calculates multiples of the standard deviation and `mean_cl_normal()` calculates the t-corrected 95% CI.

Arguments to the data function are passed to `stat_summary()`'s `fun.args` argument as a list.

The position object, `posn_d`, and the plot with jittered points, `p_wt_vs_fcyl_by_fam_jit`, are available.

```
p_wt_vs_fcyl_by_fam_jit +
  # Add a summary stat of std deviation limits
  stat_summary(fun.data = mean_sdl, 
               fun.args = list(mult = 1), 
               position = posn_d)
```

```
p_wt_vs_fcyl_by_fam_jit +
  # Change the geom to be an errorbar
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), position = posn_d, geom = "errorbar")
```

```
p_wt_vs_fcyl_by_fam_jit +
  # Add a summary stat of normal confidence limits
  stat_summary(fun.data = mean_cl_normal, position = posn_d)
```

## The coordinates layer

### Zooming In

In the video, you saw different ways of using the coordinates layer to zoom in. In this exercise, we'll compare zooming by changing scales and by changing coordinates.

The big difference is that the scale functions change the underlying dataset, which affects calculations made by computed geoms (like histograms or smooth trend lines), whereas coordinate functions make no changes to the dataset.

A scatter plot using `mtcars` with a LOESS smoothed trend line is provided. Take a look at this before updating it.

```
# Run the code, view the plot, then update it
ggplot(mtcars, aes(x = wt, y = hp, color = fam)) +
  geom_point() +
  geom_smooth() +
  # Add a continuous x scale from 3 to 6
  scale_x_continuous(limits = c(3,6))
```

Lo anterior genera un problema¡¡¡¡¡¡¡¡¡¡¡¡¡,  **SOLUCION**

```
ggplot(mtcars, aes(x = wt, y = hp, color = fam)) +
  geom_point() +
  geom_smooth() +
  # Add Cartesian coordinates with x limits from 3 to 6
  coord_cartesian(xlim = c(3,6))
```

### Aspect ratio I: 1:1 ratios

We can set the aspect ratio of a plot with [`coord_fixed()`](http://www.rdocumentation.org/packages/ggplot2/functions/coord_fixed), which uses `ratio = 1` as a default. A 1:1 aspect ratio is most appropriate when two continuous variables are on the same scale, as with the `iris` dataset.

All variables are measured in centimeters, so it only makes sense that one unit on the plot should be the same physical distance on each axis. This gives a more truthful depiction of the relationship between the two variables since the aspect ratio can change the angle of our smoothing line. This would give an erroneous impression of the data. Of course the underlying linear models don't change, but our perception can be influenced by the angle drawn.

A plot using the `iris` dataset, of sepal width vs. sepal length colored by species, is shown in the viewer.

```
ggplot(iris, aes(x = Sepal.Length, y = Sepal.Width, color = Species)) +
  geom_jitter() +
  geom_smooth(method = "lm", se = FALSE) +
  # Fix the coordinate ratio
  coord_fixed(ratio = 1)
```

```
# Fix the aspect ratio to 1:1
sun_plot +
  coord_fixed(ratio = 20)
```

Esto corrige que no se vea toda la data

```
# Change the aspect ratio to 20:1
sun_plot +
  coord_fixed(ratio = 20/1)
```

x/y

### Expand and clip

The `coord_*()` layer functions offer two useful arguments that work well together: `expand` and `clip`.

- `expand` sets a buffer margin around the plot, so data and axes don't overlap. Setting `expand` to `0` draws the axes to the limits of the data.
- `clip` decides whether plot elements that would lie outside the plot panel are displayed or ignored ("clipped").

When done properly this can make a great visual effect! We'll use `theme_classic()` and modify the axis lines in this example.

- Add Cartesian coordinates with zero expansion, to remove all buffer margins on both the x and y axes.

```
ggplot(mtcars, aes(wt, mpg)) +
  geom_point(size = 2) +
  # Add Cartesian coordinates with zero expansion
  coord_cartesian(expand = 0) +
  theme_classic()
```

```
ggplot(mtcars, aes(wt, mpg)) +
  geom_point(size = 2) +
  # Turn clipping off
  coord_cartesian(expand = 0, clip = "off") +
  theme_classic() +
  # Remove axis lines
  theme(
    axis.line = element_blank()
  )
```

## Coordinates vs Scales

## Log-transforming scales

Using `scale_y_log10()` and `scale_x_log10()` is equivalent to transforming our actual dataset *before* getting to `ggplot2`.

Using `coord_trans()`, setting `x = "log10"` and/or `y = "log10"` arguments, transforms the data *after* statistics have been calculated. The plot will look the same as with using `scale_*_log10()`, but the scales will be different, meaning that we'll see the original values on our log10 transformed axes. This can be useful since log scales can be somewhat unintuitive.

Let's see this in action with positively skewed data - the brain and body weight of 51 mammals from the `msleep` dataset.

```
# Produce a scatter plot of brainwt vs. bodywt
ggplot(msleep, aes(bodywt, brainwt)) +
  geom_point() +
  ggtitle("Raw Values")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_1.png" width ="400" >

```
# Add scale_*_*() functions
ggplot(msleep, aes(bodywt, brainwt)) +
  geom_point() +
  scale_x_log10() +
  scale_y_log10() +
  ggtitle("Scale_ functions")
```

asd

```R
# Perform a log10 coordinate system transformation
ggplot(msleep, aes(bodywt, brainwt)) +
  geom_point() +
  coord_trans(x = "log10", y = "log10")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_3.png" width ="400" >



### Adding stats to transformed scales

In the last exercise, we saw the usefulness of the `coord_trans()` function, but be careful! Remember that statistics are calculated on the untransformed data. A linear model may end up looking not-so-linear after an axis transformation. Let's revisit the two plots from the previous exercise and compare their linear models.

```R
# Plot with a scale_*_*() function:
ggplot(msleep, aes(bodywt, brainwt)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE) +
  # Add a log10 x scale
  scale_x_log10() +
  # Add a log10 y scale
  scale_y_log10() +
  ggtitle("Scale functions")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_4.png" width ="400" >

```
	# Plot with transformed coordinates
ggplot(msleep, aes(bodywt, brainwt)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE) +
  # Add a log10 coordinate transformation for x and y axes
  coord_trans(x = "log10", y = "log10")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_5.png" width ="400" >

## Double and flipped axes

### Useful double axes

Double x and y-axes are a contentious topic in data visualization. We'll revisit that discussion at the end of chapter 4. Here, I want to review a great use case where double axes actually do add value to a plot.

Our goal plot is displayed in the viewer. The two axes are the raw temperature values on a Fahrenheit scale and the transformed values on a Celsius scale.

You can imagine a similar scenar	io for Log-transformed and original values, miles and kilometers, or pounds and kilograms. A scale that is unintuitive for many people can be made easier by adding a transformation as a double axis.

```
# From previous step
y_breaks <- c(59, 68, 77, 86, 95, 104)
y_labels <- (y_breaks - 32) * 5 / 9
secondary_y_axis <- sec_axis(
  trans = identity,
  name = "Celsius",
  breaks = y_breaks,
  labels = y_labels
)

# Update the plot
ggplot(airquality, aes(Date, Temp)) +
  geom_line() +
  # Add the secondary y-axis 
  scale_y_continuous(sec.axis = secondary_y_axis) +
  labs(x = "Date (1973)", y = "Fahrenheit")
```

### Flipping axes I

*Flipping* axes means to reverse the variables mapped onto the `x` and `y` aesthetics. We can just change the mappings in `aes()`, but we can also use the `coord_flip()` layer function.

There are two reasons to use this function:

- We want a vertical geom to be horizontal, or
- We've completed a long series of plotting functions and want to flip it without having to rewrite all our commands.

```
ggplot(mtcars, aes(fcyl, fill = fam)) +
  geom_bar(position = "dodge") +
  # Flip the x and y coordinates
  coord_flip()
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_6.png" width ="400" >

```
ggplot(mtcars, aes(fcyl, fill = fam)) +
  # Set a dodge width of 0.5 for partially overlapping bars
  geom_bar(position = position_dodge(width = 0.5)) +
  coord_flip()
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_7.png" width ="400" >

### Flipping axes II

In this exercise, we'll continue to use the `coord_flip()` layer function to reverse the variables mapped onto the `x` and `y` aesthetics.

Within the `mtcars` dataset, `car` is the name of the car and `wt` is its weight.

```
# Flip the axes to set car to the y axis
ggplot(mtcars, aes(car, wt)) +
  geom_point() +
  labs(x = "car", y = "weight") +
  coord_flip()
```

## Polar coordinates

## Pie charts

The [`coord_polar()`](http://www.rdocumentation.org/packages/ggplot2/functions/coord_polar) function converts a planar x-y Cartesian plot to polar coordinates. This can be useful if you are producing pie charts.

We can imagine two forms for pie charts - the typical filled circle, or a colored ring.

Typical pie charts omit all of the non-data ink, which we saw in the themes chapter of the last course. Pie charts are not really better than stacked bar charts, but we'll come back to this point in the next chapter.

A bar plot using `mtcars` of the number of cylinders (as a factor), `fcyl`, is shown in the plot viewer.

```
# Run the code, view the plot, then update it
ggplot(mtcars, aes(x = 1, fill = fcyl)) +
  geom_bar() +
  # Add a polar coordinate system
  coord_polar(theta = "y")
```

```
ggplot(mtcars, aes(x = 1, fill = fcyl)) +
  # Reduce the bar width to 0.1
  geom_bar(width = 0.1) +
  coord_polar(theta = "y") +
  # Add a continuous x scale from 0.5 to 1.5
  scale_x_continuous(limits = c(0.5, 1.5))
```

### Wind rose plots

Polar coordinate plots are well-suited to scales like compass direction or time of day. A popular example is the "wind rose".

The `wind` dataset is taken from the `openair` package and contains hourly measurements for windspeed (`ws`) and direction (`wd`) from London in 2003. Both variables are factors.

- Use a `geom_bar()` layer, since we want to aggregate over all date values, and set the `width` argument to 1, to eliminate any spaces between the bars.

```
# Using wind, plot wd filled by ws
ggplot(wind, aes(wd, fill = ws)) +
  # Add a bar layer with width 1
  geom_bar(width = 1)
```

```
# Convert to polar coordinates:
ggplot(wind, aes(wd, fill = ws)) +
  geom_bar(width = 1) +
  coord_polar()
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_8.png" width ="400" >

```
# Convert to polar coordinates:
ggplot(wind, aes(wd, fill = ws)) +
  geom_bar(width = 1) +
  coord_polar(start = -pi/16)
```

## The facet layer

### Facet layer basics

Faceting splits the data up into groups, according to a categorical variable, then plots each group in its own panel. For splitting the data by one or two categorical variables, [`facet_grid()`](http://www.rdocumentation.org/packages/ggplot2/functions/facet_grid) is best.

Given categorical variables `A` and `B`, the code pattern is

```
plot +
  facet_grid(rows = vars(A), cols = vars(B))
```

This draws a panel for each pairwise combination of the values of `A` and `B`.

Here, we'll use the `mtcars` data set to practice. Although `cyl` and `am` are not encoded as factor variables in the data set, `ggplot2` will coerce variables to factors when used in facets.

```
ggplot(mtcars, aes(wt, mpg)) + 
  geom_point() +
  # Facet rows by am
  facet_grid(rows = vars(am))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_9.png" width ="400" >

```
ggplot(mtcars, aes(wt, mpg)) + 
  geom_point() +
  # Facet rows by am and columns by cyl
  facet_grid(rows = vars(am), cols = vars(cyl))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_10.png" width ="400" >

### Many variables

In addition to aesthetics, facets are another way of encoding factor (i.e. categorical) variables. They can be used to reduce the complexity of plots with many variables.

Our goal is the plot in the viewer, which contains 7 variables.

Two variables are mapped onto the color aesthetic, using hue and lightness. To achieve this we combined `fcyl` and `fam` into a single [interaction](https://www.rdocumentation.org/packages/base/topics/interaction) variable, `fcyl_fam`. This will allow us to take advantage of Color Brewer's *Paired* color palette.

```
# Update the plot
ggplot(mtcars, aes(x = wt, y = mpg, color = fcyl_fam, size = disp)) +
  geom_point() +
  scale_color_brewer(palette = "Paired") +
  # Grid facet on gear and vs
  facet_grid(rows = vars(gear), cols = vars(vs))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_11.png" width ="400" >

### Formula notation

As well as the `vars()` notation for specifying which variables should be used to split the dataset into facets, there is also a traditional formula notation. The three cases are shown in the table.

| Modern notation                              | Formula notation    |
| :------------------------------------------- | :------------------ |
| `facet_grid(rows = vars(A))`                 | `facet_grid(A ~ .)` |
| `facet_grid(cols = vars(B))`                 | `facet_grid(. ~ B)` |
| `facet_grid(rows = vars(A), cols = vars(B))` | `facet_grid(A ~ B)` |

`mpg_by_wt` is available again. Rework the previous plots, this time using formula notation.

```
ggplot(mtcars, aes(wt, mpg)) + 
  geom_point() +
  # Facet rows by am using formula notation
facet_grid(am ~ .)
```

Lo anterior es igual a `rows = vars(am)` 

```
ggplot(mtcars, aes(wt, mpg)) + 
  geom_point() +
  # Facet columns by cyl using formula notation
  facet_grid(. ~ cyl )
```

## Facet labels and order

### Labeling facets

If your factor levels are not clear, your facet labels may be confusing. You can assign proper labels in your original data *before* plotting (see next exercise), or you can use the `labeller` argument in the facet layer.

The default value is

- `label_value`: Default, displays only the value

Common alternatives are:

- `label_both`: Displays both the value and the variable name
- `label_context`: Displays only the values or both the values and variables depending on whether multiple factors are faceted

```
# Plot wt by mpg
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  # Label context
  facet_grid(cols = vars(cyl), labeller = label_context)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_12.png" width ="400" >

```
# Plot wt by mpg
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  # Two variables
  facet_grid(cols = vars(vs, cyl), labeller = label_context)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_13.png" width ="400" >

### Setting order

If you want to change the order of your facets, it's best to properly define your factor variables *before* plotting.

Let's see this in action with the `mtcars` transmission variable `am`. In this case, `0` = "automatic" and `1` = "manual".

Here, we'll make `am` a factor variable and relabel the numbers to proper names. The default order is alphabetical. To rearrange them we'll call `fct_rev()` from the `forcats` package to reverse the order.

```
# Make factor, set proper labels explictly
mtcars$fam <- factor(mtcars$am, labels = c(`0` = "automatic",
                                           `1` = "manual"))

# Default order is alphabetical
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  facet_grid(cols = vars(fam))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_14.png" width ="400" >

```
# Make factor, set proper labels explictly, and
# manually set the label order
mtcars$fam <- factor(mtcars$am,
                     levels = c(1, 0),
                     labels = c("manual", "automatic"))

# View again
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  facet_grid(cols = vars(fam))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_16.png" width ="400" >

## Facet plotting spaces

### Variable plotting spaces I: continuous variables

By default every facet of a plot has the same axes. If the data ranges vary wildly between facets, it can be clearer if each facet has its own scale. This is achieved with the `scales` argument to `facet_grid()`.

- `"fixed"` (default): axes are shared between facets.
- `free`: each facet has its own axes.
- `free_x`: each facet has its own x-axis, but the y-axis is shared.
- `free_y`: each facet has its own y-axis, but the x-axis is shared.

When faceting by columns, `"free_y"` has no effect, but we can adjust the x-axis. In contrast, when faceting by rows, `"free_x"` has no effect, but we can adjust the y-axis.

```
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() + 
  # Update the faceting to free the x-axis scales
  facet_grid(cols = vars(cyl), scales = "free")
```

```
ggplot(mtcars, aes(wt, mpg)) +
  geom_point() + 
  # Swap cols for rows; free the y-axis scales
  facet_grid(rows = vars(cyl), scales = "free_y")
```

### Variable plotting spaces II: categorical variables

When you have a categorical variable with many levels which are not all present in each sub-group of another variable, it's usually desirable to drop the unused levels.

By default, each facet of a plot is the same size. This behavior can be changed with the `spaces` argument, which works in the same way as `scales`: `"free_x"` allows different sized facets on the x-axis, `"free_y"`, allows different sized facets on the y-axis, `"free"` allows different sizes in both directions.

```
ggplot(mtcars, aes(x = mpg, y = car, color = fam)) +
  geom_point() +
  # Facet rows by gear
  facet_grid(rows = vars(gear))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_17.png" width ="400" >

```
ggplot(mtcars, aes(x = mpg, y = car, color = fam)) +
  geom_point() +
  # Free the y scales and space
  facet_grid(rows = vars(gear), scales = "free_y", space = "free_y")
```

## Facet wrap & margins

### Wrapping for many levels

`facet_grid()` is fantastic for categorical variables with a small number of levels. Although it is possible to facet variables with many levels, the resulting plot will be very wide or very tall, which can make it difficult to view.

The solution is to use `facet_wrap()` which separates levels along one axis but wraps all the subsets across a given number of rows or columns.

For this plot, we'll use the `Vocab` dataset that we've already seen. The base layer is provided.

Since we have many `years`, it doesn't make sense to use `facet_grid()`, so let's try `facet_wrap()` instead.

```
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  stat_smooth(method = "lm", se = FALSE) +
  # Create facets, wrapping by year, using vars()
  facet_wrap(vars(year))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_18.png" width ="400" >

```
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  stat_smooth(method = "lm", se = FALSE) +
  # Create facets, wrapping by year, using a formula
  facet_wrap(~ year)
```

```
ggplot(Vocab, aes(x = education, y = vocabulary)) +
  stat_smooth(method = "lm", se = FALSE) +
  # Update the facet layout, using 11 columns
  facet_wrap(~ year, ncol = 11)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_19.png" width ="400" >

### Margin plots

Facets are great for seeing subsets in a variable, but sometimes you want to see *both* those subsets *and* all values in a variable.

Here, the `margins` argument to `facet_grid()` is your friend.

- `FALSE` (default): no margins.
- `TRUE`: add margins to every variable being faceted by.
- `c("variable1", "variable2")`: only add margins to the variables listed.

To make it easier to follow the facets, we've created two factor variables with proper labels — `fam` for the transmission type, and `fvs` for the engine type, respectively.

*Zoom the graphics window to better view your plots.*

```
ggplot(mtcars, aes(x = wt, y = mpg)) + 
  geom_point() +
  # Facet rows by fvs and fam, and cols by gear
  facet_grid(rows = vars(fvs, fam), cols = vars(gear))
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_20.png" width ="400" >

```
ggplot(mtcars, aes(x = wt, y = mpg)) + 
  geom_point() +
  # Update the facets to only show margins on gear and fvs
  facet_grid(rows = vars(fvs, fam), cols = vars(gear), margins = "fam")
```

## Best practices: bar plots

### Bar plots: dynamite plots

In the video we saw many reasons why "dynamite plots" (bar plots with error bars) are *not* well suited for their intended purpose of depicting distributions. If you *really* want error bars on bar plots, you can of course get them, but you'll need to set the positions manually. A point geom will typically serve you much better.

Nonetheless, you should know how to handle these kinds of plots, so let's give it a try.

```
# Plot wt vs. fcyl
ggplot(mtcars, aes(x = fcyl, y = wt)) +
  # Add a bar summary stat of means, colored skyblue
  stat_summary(fun = mean, geom = "bar", fill = "skyblue") +
  # Add an errorbar summary stat std deviation limits
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), geom = "errorbar", width = 0.1)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_21.png" width ="400" >

fun = fun.y

### Bar plots: position dodging

In the previous exercise we used the `mtcars` dataset to draw a dynamite plot about the weight of the cars per cylinder type.

In this exercise we will add a distinction between transmission type, `fam`, for the dynamite plots and explore position dodging (where bars are side-by-side).

```
 # Update the aesthetics to color and fill by fam
ggplot(mtcars, aes(x = fcyl, y = wt)) +
  stat_summary(aes(color = fam, fill = fam),fun.y = mean, geom = "bar") +
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), geom = "errorbar", width = 0.1)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_22.png" width ="400" >

```
# Set alpha for the first and set position for each stat summary function
ggplot(mtcars, aes(x = fcyl, y = wt, color = fam, fill = fam)) +
  stat_summary(fun.y = mean, geom = "bar", alpha = 0.5, position = "dodge") +
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), geom = "errorbar", position = "dodge", width = 0.1)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_23.png" width ="400" >

```
# Define a dodge position object with width 0.9
posn_d <- position_dodge(width = 0.9)

# For each summary stat, update the position to posn_d
ggplot(mtcars, aes(x = fcyl, y = wt, color = fam, fill = fam)) +
  stat_summary(fun.y = mean, geom = "bar", position = posn_d, alpha = 0.5) +
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), width = 0.1, position = posn_d, geom = "errorbar")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_24.png" width ="400" >

### Bar plots: Using aggregated data

If it *is* appropriate to use bar plots (see the video!), then it nice to give an impression of the number of values in each group.

`stat_summary()` doesn't keep track of the count. [`stat_sum()`](http://www.rdocumentation.org/packages/ggplot2/functions/geom_count) does (that's the whole point), but it's difficult to access. It's more straightforward to calculate exactly what we want to plot ourselves.

Here, we've created a summary data frame called `mtcars_by_cyl` which contains the average (`mean_wt`), standard deviations (`sd_wt`) and count (`n_wt`) of car weights, for each cylinder group, `cyl`. It also contains the proportion (`prop`) of each cylinder represented in the entire dataset. Use the console to familiarize yourself with the `mtcars_by_cyl` data frame.

```
# Using mtcars_cyl, plot mean_wt vs. cyl
ggplot(mtcars_by_cyl, aes(cyl, mean_wt)) +
  # Add a bar layer with identity stat, filled skyblue
  geom_bar(stat = "identity", fill = "skyblue")
```

```
ggplot(mtcars_by_cyl, aes(x = cyl, y = mean_wt)) +
  # Set the width aesthetic to prop
  geom_col(fill = "skyblue")
```

Los anteriores dan el Mismo grafico

```
ggplot(mtcars_by_cyl, aes(x = cyl, y = mean_wt)) +
  geom_col(aes(width = prop), fill = "skyblue") +
  # Add an errorbar layer
  geom_errorbar(
    # ... at mean weight plus or minus 1 std dev
    aes(ymin = mean_wt - sd_wt, ymax = mean_wt + sd_wt),
    # with width 0.1
    width = 0.1
  )
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_26.png" width ="400" >

## Heatmaps use case scenario

### Heat maps

Since heat maps encode *color* on a continuous scale, they are difficult to accurately *decode*, a topic we discussed in the first course. Hence, heat maps are most useful if you have a small number of boxes and/or a clear pattern that allows you to overcome decoding difficulties.

To produce them, map two categorical variables onto the `x` and `y` aesthetics, along with a continuous variable onto `fill`. The [`geom_tile()`](http://www.rdocumentation.org/packages/ggplot2/functions/geom_tile) layer adds the boxes.

We'll produce the heat map we saw in the video (in the viewer) with the built-in `barley` dataset. The `barley` dataset is in the `lattice` package and has already been loaded for you. Use [`str()`](http://www.rdocumentation.org/packages/utils/functions/str) to explore the structure.

```
# Using barley, plot variety vs. year, filled by yield
ggplot(barley, aes(year, variety, fill = yield)) +
  # Add a tile geom
  geom_tile()
```

```
# Previously defined
ggplot(barley, aes(x = year, y = variety, fill = yield)) +
  geom_tile() + 
  # Facet, wrapping by site, with 1 column
  facet_wrap(facets = vars(site), ncol = 1) +
  # Add a fill scale using an 2-color gradient
  scale_fill_gradient(low = "white", high = "red")
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_27.png" width ="400" >

```
# A palette of 9 reds
red_brewer_palette <- brewer.pal(9, "Reds")

# Update the plot
ggplot(barley, aes(x = year, y = variety, fill = yield)) +
  geom_tile() + 
  facet_wrap(facets = vars(site), ncol = 1) +
  # Update scale to use n-colors from red_brewer_palette
  scale_fill_gradientn(colors = red_brewer_palette)
```

### Heat map alternatives

There are several alternatives to heat maps. The best choice really depends on the data and the story you want to tell with this data. If there is a time component, the most obvious choice is a line plot.

```
# Using barley, plot yield vs. year, colored and grouped by variety
ggplot(barley, aes(year, yield, color = variety, group = variety)) +
  # Add a line layer
  geom_line() +
  # Facet, wrapping by site, with 1 row
  facet_wrap( ~ site, nrow = 1)
```

<img src = "C:\Users\Nicol Huaraca\Desktop\Hubble\DataCamp\Ggplot2 inter\graph_28.png" width ="400" >

```
# Using barely, plot yield vs. year, colored, grouped, and filled by site
ggplot(barley, aes(x = year, y = yield, color = site, group = site, fill = site)) +
  # Add a line summary stat aggregated by mean
  stat_summary(fun.y = mean, geom = "line") +
  # Add a ribbon summary stat with 10% opacity, no color
  stat_summary(fun.data = mean_sdl, fun.args = list(mult = 1), geom = "ribbon", alpha = 0.1, color = NA)
```

## When good data makes bad plots

### Color blindness

Red-Green color blindness is surprisingly prevalent, which means that part of your audience will not be able to ready your plot if you are relying on color aesthetics.

Why would it be appropriate to use red and green in a plot?

### Typical problems

When you first encounter a data visualization, either from yourself or a colleague, you always want to critically ask if it's obscuring the data in any way.

Let's take a look at the steps we could take to produce and improve the plot in the view.

The data comes from an experiment where the effect of two different types of vitamin C sources, orange juice or ascorbic acid, were tested on the growth of the odontoblasts (cells responsible for tooth growth) in 60 guinea pigs.

The data is stored in the `TG` data frame, which contains three variables: `dose`, `len`, and `supp`.

```
# Change type
TG$dose <- as.numeric(as.character(TG$dose))

# Plot
growth_by_dose <- ggplot(TG, aes(dose, len, color = supp)) +
  stat_summary(fun.data = mean_sdl,
               fun.args = list(mult = 1),
               position = position_dodge(0.2)) +
  stat_summary(fun.y = mean,
               geom = "line",
               position = position_dodge(0.1)) +
  theme_classic() +
  # Adjust labels and colors:
  labs(x = "Dose (mg/day)", y = "Odontoblasts length (mean, standard deviation)", color = "Supplement") +
  scale_color_brewer(palette = "Set1", labels = c("Orange juice", "Ascorbic acid")) +
  scale_y_continuous(limits = c(0,35), breaks = seq(0, 35, 5), expand = c(0,0))

# View plot
growth_by_dose
```

