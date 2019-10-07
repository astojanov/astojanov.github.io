---
layout: publication
type: publication

title: "A Stage-Polymorphic IR for Compiling MATLAB-Style Dynamic Tensor Expressions"
authors: "Alen Stojanov, Tiark Rompf, Markus Püschel"
pdf: /publications/preprint/006_gpce19-mgen.pdf
conf: GPCE'19
confURL: https://conf.researchr.org/home/gpce-2019
abstract: |
           We propose a novel approach for compiling MATLAB and similar languages 
           that are characterized by tensors with dynamic shapes and types. We 
           stage an evaluator for a subset of MATLAB using the Lightweight Modular 
           Staging (LMS) framework to produce a compiler that generates C code. 
           But the first Futamura projection alone does not lead to efficient code - 
           we need to refine the rigid stage distinction based on type and shape 
           inference to remove costly runtime checks. <br />
           To this end, we introduce a stage-polymorphic data structure, that we refer to as
           <em>metacontainer</em>, to represent MATLAB tensors and their type and shape information. 
           We use metacontainers to efficiently "inject" constructs into a high-level 
           intermediate representation (IR) to infer shape and type information.  Once inferred, 
           metacontainers are also used as the primary abstraction for lowering the computation, 
           performing type, shape, and ISA specialization. Our prototype MATLAB compiler MGen 
           produces static C code that supports all primitive types, heavily overloaded operators, 
           many other dynamic aspects of the language, and explicit vectorization for SIMD architectures.
           
---