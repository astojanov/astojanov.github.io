---
layout: publication
type: publication

title: "SIMD Intrinsics on Managed Language Runtimes"
authors: "Alen Stojanov, Ivaylo Toskov, Tiark Rompf, Markus Püschel"
pdf: /publications/preprint/004_cgo18-simd.pdf
conf: CGO'18
confURL: http://cgo.org/cgo2018/
abstract: |
            Managed language runtimes such as the Java Virtual Machine (JVM) provide adequate
            performance for a wide range of applications, but at the same time, they lack much
            of the low-level control that performance-minded programmers appreciate in languages
            like C/C++. One important example is the intrinsics interface that exposes
            instructions of SIMD (Single Instruction Multiple Data) vector ISAs (Instruction Set
            Architectures). In this paper we present an automatic approach for including native
            intrinsics in the runtime of a managed language. Our implementation consists of two
            parts. First, for each vector ISA, we automatically generate the intrinsics API
            from the vendor-provided XML specification. Second, we employ a metaprogramming
            approach that enables programmers to generate and load native code at runtime. In
            this setting, programmers can use the entire high-level language as a kind of macro
            system to define new high-level vector APIs with zero overhead. As an example use
            case we show a variable precision API. We provide an end-to-end implementation of
            our approach in the HotSpot VM that supports all 5912 Intel SIMD intrinsics from
            MMX to AVX-512. Our benchmarks demonstrate that this combination of SIMD and
            metaprogramming enables developers to write high-performance, vectorized code on an
            unmodified JVM that outperforms the auto-vectorizing HotSpot just-in-time (JIT)
            compiler and provides tight integration between vectorized native code and the
            managed JVM ecosystem.
---