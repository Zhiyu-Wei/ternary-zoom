Tenary-Zoom
================
Zhiyu-Wei
2025-05-21

Main function:

``` r
draw_composition_plot <- function(data, pred, lab.title = "Title", show_boundary = FALSE,
                                  scale_color = c("yellow", "blue")) {
  # Convert input to matrix and set column names
  datapoints <- as.matrix(data)
  pred <- as.numeric(pred)
  colnames(datapoints) <- colnames(data)
  
  # Create barycentric transformation matrix
  Bmatrix <- matrix(c(0,1, 0.5,0, 0,sqrt(3)/2), 3,2)
  
  # Minimum values for each dimension (used for axis setup)
  Lmin <- min(datapoints[,1])
  Rmin <- min(datapoints[,2])
  Tmin <- min(datapoints[,3])
  sep <- (1 - Lmin - Rmin - Tmin) / 10
  
  # Triangle border in composition space
  bord.matrix <- matrix(c(1-Rmin-Tmin, Lmin, Lmin,
                          Rmin, 1-Lmin-Tmin, Rmin,
                          Tmin, Tmin, 1-Lmin-Rmin), 3, 3)
  
  # Limits based on observed data range (used for boundary lines)
  limit.matrix <- matrix(c(
    max(datapoints[,1]), Rmin, 1-max(datapoints[,1])-Rmin,
    max(datapoints[,1]), 1-max(datapoints[,1])-Tmin, Tmin,
    min(datapoints[,1]), Rmin, 1-min(datapoints[,1])-Rmin,
    min(datapoints[,1]), 1-min(datapoints[,1])-Tmin, Tmin,
    Lmin, max(datapoints[,2]), 1-Lmin-max(datapoints[,2]),
    1-max(datapoints[,2])-Tmin, max(datapoints[,2]), Tmin,
    Lmin, min(datapoints[,2]), 1-Lmin-min(datapoints[,2]),
    1-min(datapoints[,2])-Tmin, min(datapoints[,2]), Tmin,
    Lmin, 1-Lmin-max(datapoints[,3]), max(datapoints[,3]),
    1-Rmin-max(datapoints[,3]), Rmin, max(datapoints[,3]),
    Lmin, 1-Lmin-min(datapoints[,3]), min(datapoints[,3]),
    1-Rmin-min(datapoints[,3]), Rmin, min(datapoints[,3])
  ), 12, 3, byrow = TRUE)
  
  # Project compositions to 2D coordinates
  Bbord <- bord.matrix %*% Bmatrix
  Blimit <- limit.matrix %*% Bmatrix
  Bdata <- datapoints %*% Bmatrix
  
  # Main triangle and expanded outer triangle for plotting
  tar.triangle <- data.frame(
    x = c(Bbord[1,1], Bbord[2,1], Bbord[3,1], Bbord[1,1]),
    y = c(Bbord[1,2], Bbord[2,2], Bbord[3,2], Bbord[1,2])
  )
  exp.triangle <- data.frame(
    x = c(Bbord[1,1]-sep/2*sqrt(3)*2, Bbord[2,1]+sep/2*sqrt(3)*2, Bbord[3,1], Bbord[1,1]-sep/2*sqrt(3)*2),
    y = c(Bbord[1,2]-sep, Bbord[2,2]-sep, Bbord[3,2]+sep*2, Bbord[1,2]-sep)
  )
  
  # Data points projected to 2D
  point_df <- data.frame(x = Bdata[,1], y = Bdata[,2], pred = pred)
  
  # Helper function for axis tick labels
  make_axis_labels <- function(axis, coord_mat, axis_label) {
    data.frame(
      x = coord_mat[,1],
      y = coord_mat[,2],
      label = round(axis_label, 2)
    )
  }
  
  # Generate axis grid positions (L, R, T axes)
  Lpos <- matrix(c(seq(1-Rmin-Tmin, Lmin, by=-sep),
                   rep(Rmin, 11),
                   seq(Tmin, 1-Lmin-Rmin, by=sep)), 11, 3)
  Rpos <- matrix(c(seq(1-Rmin-Tmin, Lmin, by=-sep),
                   seq(Rmin, 1-Lmin-Tmin, by=sep),
                   rep(Tmin, 11)), 11, 3)
  Tpos <- matrix(c(rep(Lmin, 11),
                   seq(1-Lmin-Tmin, Rmin, by=-sep),
                   seq(Tmin, 1-Lmin-Rmin, by=sep)), 11, 3)
  
  B.Lpos <- Lpos %*% Bmatrix
  B.Rpos <- Rpos %*% Bmatrix
  B.Tpos <- Tpos %*% Bmatrix
  
  Lmarkpos <- cbind(B.Lpos[,1] - sep/10, B.Lpos[,2] + sep/10 * sqrt(3))
  Rmarkpos <- cbind(B.Rpos[,1] - sep/10, B.Rpos[,2] - sep/10 * sqrt(3))
  Tmarkpos <- cbind(B.Tpos[,1] + sep/10 * 2, B.Tpos[,2])
  
  # Tick labels for each axis
  Lname <- make_axis_labels("L", Lmarkpos, Lpos[,1])
  Rname <- make_axis_labels("R", Rmarkpos, Rpos[,2])
  Tname <- make_axis_labels("T", Tmarkpos, Tpos[,3])
  
  # Axis names (e.g., sleep / exercise / other)
  axis_labels <- data.frame(
    x = c(Bbord[1,1], Bbord[2,1], Bbord[3,1]),
    y = c(Bbord[1,2], Bbord[2,2], Bbord[3,2]),
    label = colnames(data),
    hjust = c(-1, 1, -1),
    vjust = c(-3, 3, -3),
    angle = c(54, 0, -60)
  )
  
  # Begin plot construction
  p <- ggplot() +
    geom_polygon(data = tar.triangle, aes(x, y), fill = NA, color = "steelblue1", alpha = 0.3) +
    geom_polygon(data = exp.triangle, aes(x, y), fill = NA, color = "white") +
    geom_point(data = point_df, aes(x = x, y = y, color = pred), size = 2, alpha = 0.8) +
    scale_color_gradient(low = scale_color[1], high = scale_color[2]) +
    guides(color = guide_colorbar(title = "Heart.Rate")) +
    theme_minimal() +
    labs(title = lab.title, x = "", y = "") +
    theme(axis.text = element_blank(), axis.ticks = element_blank(), panel.grid = element_blank()) +
    geom_text(data = axis_labels, aes(x = x, y = y, label = label),
              size = 5, hjust = axis_labels$hjust,
              vjust = axis_labels$vjust, angle = axis_labels$angle) +
    geom_segment(aes(x = B.Lpos[,1], y = B.Lpos[,2], xend = Lmarkpos[,1], yend = Lmarkpos[,2]), color = "lightblue") +
    geom_segment(aes(x = B.Rpos[,1], y = B.Rpos[,2], xend = Rmarkpos[,1], yend = Rmarkpos[,2]), color = "lightblue") +
    geom_segment(aes(x = B.Tpos[,1], y = B.Tpos[,2], xend = Tmarkpos[,1], yend = Tmarkpos[,2]), color = "lightblue") +
    geom_text(data = Lname, aes(x, y, label = label), size = 2, hjust = 1, vjust = 0, angle = -60) +
    geom_text(data = Rname, aes(x, y, label = label), size = 2, hjust = 1, vjust = 0, angle = 60) +
    geom_text(data = Tname, aes(x, y, label = label), size = 2, hjust = 0, vjust = 0, angle = 0) +
    geom_segment(aes(x = B.Lpos[2:10,1], y = B.Lpos[2:10,2], xend = B.Rpos[2:10,1], yend = B.Rpos[2:10,2]),
                 color = "lightblue", linetype = 2, alpha = 0.3) +
    geom_segment(aes(x = B.Rpos[2:10,1], y = B.Rpos[2:10,2], xend = B.Tpos[10:2,1], yend = B.Tpos[10:2,2]),
                 color = "lightblue", linetype = 2, alpha = 0.3) +
    geom_segment(aes(x = B.Tpos[2:10,1], y = B.Tpos[2:10,2], xend = B.Lpos[2:10,1], yend = B.Lpos[2:10,2]),
                 color = "lightblue", linetype = 2, alpha = 0.3)
  
  # Optionally show data bounds
  if (show_boundary) {
    boundary_df <- data.frame(
      x = Blimit[seq(1, 11, 2), 1],
      y = Blimit[seq(1, 11, 2), 2],
      xend = Blimit[seq(2, 12, 2), 1],
      yend = Blimit[seq(2, 12, 2), 2]
    )
    p <- p + geom_segment(data = boundary_df,
                          aes(x = x, y = y, xend = xend, yend = yend),
                          color = "red")
  }
  
  return(p)
}
```

📂 Data Description and Preparation

In this example, we aim to visualize how a three-part composition—Sleep,
Exercise, and Others—relates to a physiological outcome using a custom
ternary plot.

The dataset contains both compositional variables (proportions that sum
to 1) and non-compositional variables (e.g., age, gender, physiological
measurements). We also include a target variable (e.g., heart rate) as
the response to be visualized in color.

This dataset is from Kaggle:
<https://www.kaggle.com/datasets/uom190346a/sleep-health-and-lifestyle-dataset>

Before plotting, we:

Read the dataset from a CSV file.

Rename the three compositional columns for clarity.

Split the data into:

data2: the compositional matrix (Sleep / Exercise / Others)

data1: other covariates

obj: the response variable to be visualized

The compositional part (data2) must be explicitly converted into a
matrix, as required by the plotting function.

``` r
finaldata=read.csv("C:\\Users\\first\\OneDrive\\桌面\\paper\\finaldata.csv")
finaldata=finaldata[,-1]
colnames(finaldata)[12:14]=c("Sleep","Exercise","Others")


obj=finaldata[,7]
data1=finaldata[,c(1,2,4:6,8:11)]
data2=as.matrix(finaldata[,c(12:14)]) 
```

use ggtern package to plot the ternary plot, we can see most of the data
points are concentrated in the lower right corner.

``` r
library(ggtern)
df=data.frame(
  Sleep = data2[,1],
  Exercise = data2[,2],
  Others = data2[,3],
  Heart.Rate = obj
)

ggtern(data = df, aes(x = Sleep, y =Others, z =Exercise )) +
  geom_point(aes(color = obj), size = 3) +
  labs(title = "Ternary Plot") +
  theme(legend.position = "right") +
  scale_color_gradient(low = "green", high = "red",
                       name = "Heart Rate")
```

![](Zoom-Ternary_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
detach("package:ggtern", unload = TRUE)
```

Remember to detach the ggtern package after use to avoid conflicts with
ggplot2 packages.

Now, let’s take a look at the output of draw_composition_plot.

``` r
library(ggplot2)
draw_composition_plot(data2,obj,lab.title = "Zoom Ternary",
                      show_boundary = TRUE,scale_color = c("green", "red"))
```

![](Zoom-Ternary_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->
