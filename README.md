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


