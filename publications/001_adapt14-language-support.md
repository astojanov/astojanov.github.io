---
layout: publication
type: publication

title: "Language Support for the Construction of High Performance Code Generators"
authors: "Georg Ofenbeck, Tiark Rompf, Alen Stojanov, Martin Odersky and Markus Püschel"
pdf: /publications/preprint/001_adapt14-language-support.pdf
conf: ADAPT'14
confURL: http://adapt-workshop.org/2014/
abstract: |
            The development of highest performance code on modern processors is extremely difficult due to deep memory hierarchies,
            vector instructions, multiple cores, and inherent limitations of compilers. The problem is particularly noticeable for
            library functions of mathematical nature (e.g., BLAS, FFT, filters, Viterbi decoders) that are performance-critical in
            areas such as multimedia processing, computer vision, graphics, machine learning, or scientific computing. Experience
            shows that a straightforward implementation often underperforms by one or two orders of magnitude compared to highly
            tuned code. The latter is often highly specialized to a platform which makes porting very costly. One appealing solution
            to the problem of optimizing and porting libraries are program generators that automatically produce highest performance
            libraries for a given platform from a high level description. Building such a generator is difficult, which is the reason
            that only very few exist to date. The difficulty comes from both the problem of designing an extensible approach to perform
            all the optimizations the compiler is unable to do and the actual implementation of the generator. To tackle the latter
            problem we asked the question: Which programming languages features support the development of generators for performance
            libraries?
---
