---
layout: page
title: AR Sudoku Solver
permalink: /projects/arsudoku
---

The application detects Sudoku grids in the camera image stream, solves the Sudoku and superimposes the result on top of the image. The detection first locates the gird corners and uses them to undo the perspective warping of the Sudoku. The projected image is split into 81 cell patches. For A small neural network is used for digit classification.

![Example output of the Sudoku solver](/assets/images/sudoku_example.jpg)

Since the application is a very simple prototype there are many flaws in its implementation. For one, the detection algorithm could be faster, especially if the image binarization yields a suboptimal result with a lot of noise. Additionally, the digit classification network was trained on a set of hand-written digits instead of actual sample data produced by the application. Therefore, there are some differences between training and test time. This leads to classification errors and hence to unsolvable Sudokus or wrong results. Furthermore, the rendering pipeline and the detection steps are closely coupled, thus the preview image has some lag because the detection cannot reach a high enough throughput.

Code on [GitHub](https://github.com/JBamberger/sudoku).
