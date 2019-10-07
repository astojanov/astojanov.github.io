---
layout: publication
type: publication

title: "Fast Quantized Arithmetic on x86: Trading Compute for Data Movement"
authors: "Alen Stojanov, Tyler Michael Smith, Dan Alistarh, Markus Püschel"
pdf: /publications/preprint/005_sips18-clover.pdf
conf: SIPS'18
confURL: http://sites.ieee.org/sips2018/
abstract: |
We introduce a new library for the efficient computation on
               low-precision data, providing mathematical routines required
               by fundamental methods in optimization and sparse recovery.
               Our library faithfully implements variants of stochastic
               quantization that guarantee convergence at low precision,
               and supports data formats from 4-bit quantized to 32-bit
               IEEE-754 on current Intel processors. In particular, we
               show that 4-bit can be implemented efficiently using Intel
               AVX despite the lack of native support for this data format.
               Experimental results with dot product, matrix-vector
               multiplication, gradient descent (GD), and iterative hard
               thresholding (IHT) illustrate the attainable speedups,
               in many cases close to linear in the reduction of precision
               because of reduced data movement. Finally, for GD and IHT
               we show examples of absolute speedup achieved by 4-bit
               versus 32-bit by iterating until a given target error is achieved.
   ---