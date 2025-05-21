# 🎯 focused-ternary-plot

A flexible and focused ternary plot function for compositional data visualization in R.

Unlike traditional ternary plotting tools like [`ggtern`](https://github.com/hdrake/ggtern), which always show the full triangle, this function adapts the plot area to match **the actual data region**. It is especially useful when your compositions are concentrated in a small area of the triangle.

---

## ✨ Features

- ✅ Works with any 3-part compositional data (e.g., sleep/exercise/other, macronutrients, soil types)
- ✅ Automatically adapts the triangle to your data's support
- ✅ Optional red boundary lines show the exact area where data exists
- ✅ Color mapping for predicted or measured values
- ✅ Fully `ggplot2`-based — no need for external packages like `ggtern`

---

🧭 Why not use ggtern?
The ggtern package is great, but it has one major limitation:

It always displays the entire triangle, even when your data occupies only a small region.

This function solves that by:

Zooming into the actual data region

Making patterns and clusters easier to see

Saving space in reports or dashboards


🔧 Arguments

draw_composition_plot(
  data,             # 3-column composition data.frame or matrix
  pred,             # numeric response vector
  lab.title = "",   # plot title
  show_boundary = FALSE,     # whether to draw data boundaries
  scale_color = c("yellow", "blue")  # gradient for response values
)


📜 License
MIT License. Feel free to use, modify, and share.




