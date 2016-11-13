---
layout: publication
type: publication

title: "Spiral in Scala: Towards the Systematic Construction of Generators for Performance Libraries"
authors: "Georg Ofenbeck, Tiark Rompf, Alen Stojanov, Martin Odersky and Markus Püschel"
pdf: http://spiral.ece.cmu.edu:8080/pub-spiral/pubfile/paper_170.pdf
conf: GPCE'13
confURL: http://program-transformation.org/GPCE13/WebHome
abstract: |
            Program generators for high performance libraries are an appealing solution to the recurring problem of porting and optimizing
            code with every new processor generation, but only few such generators exist to date. This is due to not only the difficulty
            of the design, but also of the actual implementation, which often results in an ad-hoc collection of standalone programs and
            scripts that are hard to extend, maintain, or reuse. In this paper we ask whether and which programming language concepts and
            features are needed to enable a more systematic construction of such generators. The systematic approach we advocate extrapolates
            from existing generators: a) describing the problem and algorithmic knowledge using one, or several, domain-specific
            languages (DSLs), b) expressing optimizations and choices as rewrite rules on DSL programs, c) designing data structures that can
            be configured to control the type of code that is generated and the data representation used, and d) using autotuning to select
            the best-performing alternative. As a case study, we implement a small, but representative subset of Spiral in Scala using the
            Lightweight Modular Staging (LMS) framework. The first main contribution of this paper is the realization of c) using type classes
            to abstract over staging decisions, i.e. which pieces of a computation are performed immediately and for which pieces code is
            generated. Specifically, we abstract over different complex data representations jointly with different code representations
            including generating loops versus unrolled code with scalar replacement---a crucial and usually tedious performance transformation.
            The second main contribution is to provide full support for a) and d) within the LMS framework: we extend LMS to support translation
            between different DSLs and autotuning through search.
---
