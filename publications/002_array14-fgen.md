---
layout: publication
type: publication

title: "Abstracting Vector Architectures in Library Generators: Case Study Convolution Filters"
authors: "Alen Stojanov, Georg Ofenbeck, Tiark Rompf, Markus Puschel"
pdf: http://spiral.ece.cmu.edu:8080/pub-spiral/pubfile/paper_179.pdf
conf: ARRAY'14
confURL: http://snapl.org/2015/index.html
abstract: |
          We present FGen, a program generator for high performance convolution operations (finite-impulse-response filters).
          The generator uses an internal mathematical DSL to enable structural optimization at a high level of abstraction.
          We use FGen as a testbed to demonstrate how to provide modular and extensible support for modern SIMD vector
          architectures in a DSL-based generator. Specifically, we show how to combine staging and generic programming with
          type classes to abstract over both the data type (real or complex) and the target architecture (e.g., SSE or AVX)
          when mapping DSL expressions to C code with explicit vector intrinsics. Benchmarks shows that the generated code
          is highly competitive with commercial libraries.
---

