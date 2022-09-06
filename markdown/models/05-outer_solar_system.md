---
author: "Yingbo Ma, Chris Rackauckas"
title: "The Outer Solar System"
---


## Data


The chosen units are: masses relative to the sun, so that the sun has mass $1$. We have taken $m_0 = 1.00000597682$ to take account of the inner planets. Distances are in astronomical units , times in earth days, and the gravitational constant is thus $G = 2.95912208286 \cdot 10^{-4}$.

| planet | mass | initial position | initial velocity |
| --- | --- | --- | --- |
| Jupiter | $m_1 = 0.000954786104043$ | <ul><li>-3.5023653</li><li>-3.8169847</li><li>-1.5507963</li></ul> | <ul><li>0.00565429</li><li>-0.00412490</li><li>-0.00190589</li></ul>
| Saturn | $m_2 = 0.000285583733151$ | <ul><li>9.0755314</li><li>-3.0458353</li><li>-1.6483708</li></ul> | <ul><li>0.00168318</li><li>0.00483525</li><li>0.00192462</li></ul>
| Uranus | $m_3 = 0.0000437273164546$ | <ul><li>8.3101420</li><li>-16.2901086</li><li>-7.2521278</li></ul> | <ul><li>0.00354178</li><li>0.00137102</li><li>0.00055029</li></ul>
| Neptune | $m_4 = 0.0000517759138449$ | <ul><li>11.4707666</li><li>-25.7294829</li><li>-10.8169456</li></ul> | <ul><li>0.00288930</li><li>0.00114527</li><li>0.00039677</li></ul>
| Pluto | $ m_5 = 1/(1.3 \cdot 10^8 )$ | <ul><li>-15.5387357</li><li>-25.2225594</li><li>-3.1902382</li></ul> | <ul><li>0.00276725</li><li>-0.00170702</li><li>-0.00136504</li></ul>

The data is taken from the book "Geometric Numerical Integration" by E. Hairer, C. Lubich and G. Wanner.

```julia
using Plots, OrdinaryDiffEq, ModelingToolkit
gr()

G = 2.95912208286e-4
M = [1.00000597682, 0.000954786104043, 0.000285583733151, 0.0000437273164546, 0.0000517759138449, 1/1.3e8]
planets = ["Sun", "Jupiter", "Saturn", "Uranus", "Neptune", "Pluto"]

pos =  [0.0  -3.5023653   9.0755314    8.310142    11.4707666  -15.5387357
        0.0  -3.8169847  -3.0458353  -16.2901086  -25.7294829  -25.2225594
        0.0  -1.5507963  -1.6483708   -7.2521278  -10.8169456   -3.1902382]
vel = [0.0   0.00565429  0.00168318  0.00354178  0.0028893    0.00276725
       0.0  -0.0041249   0.00483525  0.00137102  0.00114527  -0.00170702
       0.0  -0.00190589  0.00192462  0.00055029  0.00039677  -0.00136504]
tspan = (0.0, 200_000.0)
```

```
(0.0, 200000.0)
```





The N-body problem's Hamiltonian is

$$H(p,q) = \frac{1}{2}\sum_{i=0}^{N}\frac{p_{i}^{T}p_{i}}{m_{i}} - G\sum_{i=1}^{N}\sum_{j=0}^{i-1}\frac{m_{i}m_{j}}{\left\lVert q_{i}-q_{j} \right\rVert}$$

Here, we want to solve for the motion of the five outer planets relative to the sun, namely, Jupiter, Saturn, Uranus, Neptune and Pluto.

```julia
const ∑ = sum
const N = 6
@variables t u(t)[1:3, 1:N]
u = collect(u)
D = Differential(t)
potential = -G*∑(i->∑(j->(M[i]*M[j])/√(∑(k->(u[k, i]-u[k, j])^2, 1:3)), 1:i-1), 2:N)
```

```
-2.8253455313622585e-7 / sqrt(((u(t))[1, 2] - (u(t))[1, 1])^2 + ((u(t))[2, 
2] - (u(t))[2, 1])^2 + ((u(t))[3, 2] - (u(t))[3, 1])^2) + -8.45082182146621
2e-8 / sqrt(((u(t))[1, 3] - (u(t))[1, 1])^2 + ((u(t))[2, 3] - (u(t))[2, 1])
^2 + ((u(t))[3, 3] - (u(t))[3, 1])^2) + -1.2939524111245703e-8 / sqrt(((u(t
))[1, 4] - (u(t))[1, 1])^2 + ((u(t))[2, 4] - (u(t))[2, 1])^2 + ((u(t))[3, 4
] - (u(t))[3, 1])^2) + -8.068679017837167e-11 / sqrt(((u(t))[1, 3] - (u(t))
[1, 2])^2 + ((u(t))[2, 3] - (u(t))[2, 2])^2 + ((u(t))[3, 3] - (u(t))[3, 2])
^2) + -1.2354403974297985e-11 / sqrt(((u(t))[1, 4] - (u(t))[1, 2])^2 + ((u(
t))[2, 4] - (u(t))[2, 2])^2 + ((u(t))[3, 4] - (u(t))[3, 2])^2) + -1.5321216
573476373e-8 / sqrt(((u(t))[1, 5] - (u(t))[1, 1])^2 + ((u(t))[2, 5] - (u(t)
)[2, 1])^2 + ((u(t))[3, 5] - (u(t))[3, 1])^2) + -2.2762613607692674e-12 / s
qrt(((u(t))[1, 6] - (u(t))[1, 1])^2 + ((u(t))[2, 6] - (u(t))[2, 1])^2 + ((u
(t))[3, 6] - (u(t))[3, 1])^2) + -1.4628397250091296e-11 / sqrt(((u(t))[1, 5
] - (u(t))[1, 2])^2 + ((u(t))[2, 5] - (u(t))[2, 2])^2 + ((u(t))[3, 5] - (u(
t))[3, 2])^2) + -2.1733297268319285e-15 / sqrt(((u(t))[1, 6] - (u(t))[1, 2]
)^2 + ((u(t))[2, 6] - (u(t))[2, 2])^2 + ((u(t))[3, 6] - (u(t))[3, 2])^2) + 
-3.695295514770784e-12 / sqrt(((u(t))[1, 4] - (u(t))[1, 3])^2 + ((u(t))[2, 
4] - (u(t))[2, 3])^2 + ((u(t))[3, 4] - (u(t))[3, 3])^2) + -4.37546407410716
75e-12 / sqrt(((u(t))[1, 5] - (u(t))[1, 3])^2 + ((u(t))[2, 5] - (u(t))[2, 3
])^2 + ((u(t))[3, 5] - (u(t))[3, 3])^2) + -6.500593317482474e-16 / sqrt(((u
(t))[1, 6] - (u(t))[1, 3])^2 + ((u(t))[2, 6] - (u(t))[2, 3])^2 + ((u(t))[3,
 6] - (u(t))[3, 3])^2) + -6.699516813972553e-13 / sqrt(((u(t))[1, 5] - (u(t
))[1, 4])^2 + ((u(t))[2, 5] - (u(t))[2, 4])^2 + ((u(t))[3, 5] - (u(t))[3, 4
])^2) + -9.953420595770331e-17 / sqrt(((u(t))[1, 6] - (u(t))[1, 4])^2 + ((u
(t))[2, 6] - (u(t))[2, 4])^2 + ((u(t))[3, 6] - (u(t))[3, 4])^2) + -1.178548
077066926e-16 / sqrt(((u(t))[1, 6] - (u(t))[1, 5])^2 + ((u(t))[2, 6] - (u(t
))[2, 5])^2 + ((u(t))[3, 6] - (u(t))[3, 5])^2)
```





## Hamiltonian System

`NBodyProblem` constructs a second order ODE problem under the hood. We know that a Hamiltonian system has the form of

$$\dot{p} = -H_{q}(p,q)\quad \dot{q}=H_{p}(p,q)$$

For an N-body system, we can symplify this as:

$$\dot{p} = -\nabla{V}(q)\quad \dot{q}=M^{-1}p.$$

Thus $\dot{q}$ is defined by the masses. We only need to define $\dot{p}$, and this is done internally by taking the gradient of $V$. Therefore, we only need to pass the potential function and the rest is taken care of.

```julia
eqs = vec(@. D(D(u))) .~ .- ModelingToolkit.gradient(potential, vec(u)) ./ repeat(M, inner=3)
@named sys = ODESystem(eqs, t)
ss = structural_simplify(sys)
prob = ODEProblem(ss, [vec(u .=> pos); vec(D.(u) .=> vel)], tspan)
sol = solve(prob, Tsit5());
```


```julia
plt = plot()
for i in 1:N
    plot!(plt, sol, idxs=(u[:, i]...,), lab = planets[i])
end
plot!(plt; xlab = "x", ylab = "y", zlab = "z", title = "Outer solar system")
```

![](figures/05-outer_solar_system_4_1.png)


## Appendix

These tutorials are a part of the SciMLTutorials.jl repository, found at: [https://github.com/SciML/SciMLTutorials.jl](https://github.com/SciML/SciMLTutorials.jl). For more information on high-performance scientific machine learning, check out the SciML Open Source Software Organization [https://sciml.ai](https://sciml.ai).

To locally run this tutorial, do the following commands:

```
using SciMLTutorials
SciMLTutorials.weave_file("tutorials/models","05-outer_solar_system.jmd")
```

Computer Information:

```
Julia Version 1.8.0
Commit 5544a0fab76 (2022-08-17 13:38 UTC)
Platform Info:
  OS: Linux (x86_64-linux-gnu)
  CPU: 128 × AMD EPYC 7502 32-Core Processor
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-13.0.1 (ORCJIT, znver2)
  Threads: 1 on 128 virtual cores
Environment:
  JULIA_CPU_THREADS = 128
  JULIA_DEPOT_PATH = /cache/julia-buildkite-plugin/depots/a6029d3a-f78b-41ea-bc97-28aa57c6c6ea

```

Package Information:

```
Status `/cache/build/exclusive-amdci1-0/julialang/scimltutorials-dot-jl/tutorials/models/Project.toml`
  [479239e8] Catalyst v12.2.1
  [459566f4] DiffEqCallbacks v2.24.1
  [f3b72e0c] DiffEqDevTools v2.31.2
  [055956cb] DiffEqPhysics v3.9.0
  [0c46a032] DifferentialEquations v7.3.0
  [31c24e10] Distributions v0.25.70
  [587475ba] Flux v0.13.5
  [f6369f11] ForwardDiff v0.10.32
  [23fbe1c1] Latexify v0.15.16
  [961ee093] ModelingToolkit v8.21.0
  [2774e3e8] NLsolve v4.5.1
⌅ [315f7962] NeuralPDE v4.11.0
  [429524aa] Optim v1.7.2
  [1dea7af3] OrdinaryDiffEq v6.26.2
  [91a5bcdd] Plots v1.32.0
  [731186ca] RecursiveArrayTools v2.32.0
  [30cb0354] SciMLTutorials v1.0.0
  [789caeaf] StochasticDiffEq v6.53.0
  [37e2e46d] LinearAlgebra
  [2f01184e] SparseArrays
Info Packages marked with ⌅ have new versions available but cannot be upgraded. To see why use `status --outdated`
```

And the full manifest:

```
Status `/cache/build/exclusive-amdci1-0/julialang/scimltutorials-dot-jl/tutorials/models/Manifest.toml`
  [c3fe647b] AbstractAlgebra v0.27.4
  [621f4979] AbstractFFTs v1.2.1
  [1520ce14] AbstractTrees v0.4.2
  [7d9f7c33] Accessors v0.1.20
  [79e6a3ab] Adapt v3.4.0
  [dce04be8] ArgCheck v2.3.0
  [ec485272] ArnoldiMethod v0.2.0
  [4fba245c] ArrayInterface v6.0.23
  [30b0a656] ArrayInterfaceCore v0.1.20
  [6ba088a2] ArrayInterfaceGPUArrays v0.2.1
  [015c0d05] ArrayInterfaceOffsetArrays v0.1.6
  [b0d46f97] ArrayInterfaceStaticArrays v0.1.4
  [dd5226c6] ArrayInterfaceStaticArraysCore v0.1.0
  [a2b0951a] ArrayInterfaceTracker v0.1.1
  [4c555306] ArrayLayouts v0.8.11
  [15f4f7f2] AutoHashEquals v0.2.0
  [ab4f0b2a] BFloat16s v0.2.0
  [aae01518] BandedMatrices v0.17.6
  [198e06fe] BangBang v0.3.36
  [9718e550] Baselet v0.1.1
  [e2ed5e7c] Bijections v0.1.4
  [62783981] BitTwiddlingConvenienceFunctions v0.1.4
  [8e7c35d0] BlockArrays v0.16.20
  [ffab5731] BlockBandedMatrices v0.11.9
  [764a87c0] BoundaryValueDiffEq v2.9.0
  [fa961155] CEnum v0.4.2
  [2a0fbf3d] CPUSummary v0.1.25
  [00ebfdb7] CSTParser v3.3.6
  [052768ef] CUDA v3.12.0
  [49dc2e85] Calculus v0.5.1
  [7057c7e9] Cassette v0.3.10
  [479239e8] Catalyst v12.2.1
  [082447d4] ChainRules v1.44.6
  [d360d2e6] ChainRulesCore v1.15.4
  [9e997f8a] ChangesOfVariables v0.1.4
  [fb6a15b2] CloseOpenIntervals v0.1.10
  [944b1d66] CodecZlib v0.7.0
  [35d6a980] ColorSchemes v3.19.0
  [3da002f7] ColorTypes v0.11.4
  [c3611d14] ColorVectorSpace v0.9.9
  [5ae59095] Colors v0.12.8
  [861a8166] Combinatorics v1.0.2
  [a80b9123] CommonMark v0.8.6
  [38540f10] CommonSolve v0.2.1
  [bbf7d656] CommonSubexpressions v0.3.0
⌅ [34da2185] Compat v3.46.0
  [b0b7db55] ComponentArrays v0.13.2
  [b152e2b5] CompositeTypes v0.1.2
  [a33af91c] CompositionsBase v0.1.1
  [8f4d0f93] Conda v1.7.0
  [88cd18e8] ConsoleProgressMonitor v0.1.2
  [187b0558] ConstructionBase v1.4.1
  [6add18c4] ContextVariablesX v0.1.2
  [d38c429a] Contour v0.6.2
  [adafc99b] CpuId v0.3.1
  [a8cc5b0e] Crayons v4.1.1
  [8a292aeb] Cuba v2.2.0
  [667455a9] Cubature v1.5.1
  [9a962f9c] DataAPI v1.10.0
  [82cc6244] DataInterpolations v3.10.1
  [864edb3b] DataStructures v0.18.13
  [e2d170a0] DataValueInterfaces v1.0.0
  [244e2a9f] DefineSingletons v0.1.2
  [bcd4f6db] DelayDiffEq v5.37.1
  [b429d917] DensityInterface v0.4.0
  [2b5f629d] DiffEqBase v6.100.0
  [459566f4] DiffEqCallbacks v2.24.1
  [f3b72e0c] DiffEqDevTools v2.31.2
  [aae7a2af] DiffEqFlux v1.52.0
  [77a26b50] DiffEqNoiseProcess v5.12.3
  [9fdde737] DiffEqOperators v4.43.1
  [055956cb] DiffEqPhysics v3.9.0
  [41bf760c] DiffEqSensitivity v6.79.0
  [163ba53b] DiffResults v1.0.3
  [b552c78f] DiffRules v1.11.1
  [0c46a032] DifferentialEquations v7.3.0
  [b4f34e82] Distances v0.10.7
  [31c24e10] Distributions v0.25.70
  [ced4e74d] DistributionsAD v0.6.42
⌅ [ffbed154] DocStringExtensions v0.8.6
  [5b8099bc] DomainSets v0.5.13
  [fa6b7ba4] DualNumbers v0.6.8
  [7c1d4256] DynamicPolynomials v0.4.5
  [da5c29d0] EllipsisNotation v1.6.0
  [7da242da] Enzyme v0.10.4
  [d4d017d3] ExponentialUtilities v1.18.0
  [e2ba6199] ExprTools v0.1.8
  [411431e0] Extents v0.1.1
  [c87230d0] FFMPEG v0.4.1
  [cc61a311] FLoops v0.2.0
  [b9860ae5] FLoopsBase v0.1.1
  [7034ab61] FastBroadcast v0.2.1
  [9aa1b823] FastClosures v0.3.2
  [29a986be] FastLapackInterface v1.2.6
  [1a297f60] FillArrays v0.13.4
⌃ [6a86dc24] FiniteDiff v2.13.1
  [53c48c17] FixedPointNumbers v0.8.4
  [587475ba] Flux v0.13.5
  [9c68100b] FoldsThreads v0.1.1
  [59287772] Formatting v0.4.2
  [f6369f11] ForwardDiff v0.10.32
  [069b7b12] FunctionWrappers v1.1.2
  [77dc65aa] FunctionWrappersWrappers v0.1.1
  [d9f16b24] Functors v0.3.0
  [0c68f7d7] GPUArrays v8.5.0
  [46192b85] GPUArraysCore v0.1.2
  [61eb1bfa] GPUCompiler v0.16.3
  [28b8d3ca] GR v0.66.2
  [c145ed77] GenericSchur v0.5.3
  [cf35fbd7] GeoInterface v1.0.1
  [5c1252a2] GeometryBasics v0.4.3
  [86223c79] Graphs v1.7.2
  [42e2da0e] Grisu v1.0.2
  [0b43b601] Groebner v0.2.10
  [d5909c97] GroupsCore v0.4.0
  [19dc6840] HCubature v1.5.0
  [cd3eb016] HTTP v1.3.3
⌅ [eafb193a] Highlights v0.4.5
  [3e5b6fbb] HostCPUFeatures v0.1.8
  [34004b35] HypergeometricFunctions v0.3.11
  [7073ff75] IJulia v1.23.3
  [7869d1d1] IRTools v0.4.6
  [615f187c] IfElse v0.1.1
  [d25df0c9] Inflate v0.1.3
  [83e8ac13] IniFile v0.5.1
  [22cec73e] InitialValues v0.3.1
  [18e54dd8] IntegerMathUtils v0.1.0
  [8197267c] IntervalSets v0.7.2
  [3587e190] InverseFunctions v0.1.7
  [92d709cd] IrrationalConstants v0.1.1
  [c8e1da08] IterTools v1.4.0
  [42fd0dbc] IterativeSolvers v0.9.2
  [82899510] IteratorInterfaceExtensions v1.0.0
  [692b3bcd] JLLWrappers v1.4.1
  [682c06a0] JSON v0.21.3
  [98e50ef6] JuliaFormatter v1.0.9
  [b14d175d] JuliaVariables v0.2.4
  [ccbc3e58] JumpProcesses v9.2.0
  [ef3ab10e] KLU v0.3.0
  [ba0b0d4f] Krylov v0.8.3
  [0b1a1467] KrylovKit v0.5.4
  [929cbde3] LLVM v4.14.0
  [b964fa9f] LaTeXStrings v1.3.0
  [2ee39098] LabelledArrays v1.12.0
  [23fbe1c1] Latexify v0.15.16
  [a5e1c1ea] LatinHypercubeSampling v1.8.0
  [73f95e8e] LatticeRules v0.0.1
  [10f19ff3] LayoutPointers v0.1.10
  [50d2b5c4] Lazy v0.15.1
  [5078a376] LazyArrays v0.22.11
⌅ [d7e5e226] LazyBandedMatrices v0.7.17
  [0fc2ff8b] LeastSquaresOptim v0.8.3
  [1d6d02ad] LeftChildRightSiblingTrees v0.2.0
  [2d8b4e74] LevyArea v1.0.0
  [d3d80556] LineSearches v7.2.0
  [7ed4a6bd] LinearSolve v1.26.0
  [2ab3a3ac] LogExpFunctions v0.3.18
  [e6f89c97] LoggingExtras v0.4.9
  [bdcacae8] LoopVectorization v0.12.125
  [b2108857] Lux v0.4.21
  [d8e11817] MLStyle v0.4.13
  [f1d291b0] MLUtils v0.2.10
  [1914dd2f] MacroTools v0.5.9
  [d125e4d3] ManualMemory v0.1.8
  [a3b82374] MatrixFactorizations v0.9.2
  [739be429] MbedTLS v1.1.5
  [442fdcdd] Measures v0.3.1
  [c03570c3] Memoize v0.4.4
  [e9d8d322] Metatheory v1.3.4
  [128add7d] MicroCollections v0.1.2
  [e1d29d7a] Missings v1.0.2
  [961ee093] ModelingToolkit v8.21.0
⌅ [4886b29c] MonteCarloIntegration v0.0.3
  [46d2c3a1] MuladdMacro v0.2.2
  [102ac46a] MultivariatePolynomials v0.4.6
  [ffc61752] Mustache v1.0.14
  [d8a4904e] MutableArithmetics v1.0.4
  [d41bc354] NLSolversBase v7.8.2
  [2774e3e8] NLsolve v4.5.1
  [872c559c] NNlib v0.8.9
  [a00861dc] NNlibCUDA v0.2.4
  [77ba4419] NaNMath v1.0.1
  [71a1bf82] NameResolution v0.1.5
⌅ [315f7962] NeuralPDE v4.11.0
  [8913a72c] NonlinearSolve v0.3.22
  [d8793406] ObjectFile v0.3.7
  [6fe1bfb0] OffsetArrays v1.12.7
  [429524aa] Optim v1.7.2
  [3bd65402] Optimisers v0.2.9
  [7f7a1694] Optimization v3.8.2
  [253f991c] OptimizationFlux v0.1.0
  [36348300] OptimizationOptimJL v0.1.2
  [42dfb2eb] OptimizationOptimisers v0.1.0
  [500b13db] OptimizationPolyalgorithms v0.1.0
  [bac558e1] OrderedCollections v1.4.1
  [1dea7af3] OrdinaryDiffEq v6.26.2
  [90014a1f] PDMats v0.11.16
  [d96e819e] Parameters v0.12.3
  [69de0a69] Parsers v2.4.0
  [ccf2f8ad] PlotThemes v3.0.0
  [995b91a9] PlotUtils v1.3.0
  [91a5bcdd] Plots v1.32.0
  [e409e4f3] PoissonRandom v0.4.1
  [f517fe37] Polyester v0.6.15
  [1d0040c9] PolyesterWeave v0.1.9
  [85a6dd25] PositiveFactorizations v0.2.4
  [d236fae5] PreallocationTools v0.4.2
  [21216c6a] Preferences v1.3.0
  [8162dcfd] PrettyPrint v0.2.0
  [27ebfcd6] Primes v0.5.3
  [33c8b6b6] ProgressLogging v0.1.4
  [92933f4c] ProgressMeter v1.7.2
  [1fd47b50] QuadGK v2.5.0
  [67601950] Quadrature v2.1.0
  [e0ec9b62] QuadratureCubature v0.1.1
  [8a4e6c94] QuasiMonteCarlo v0.2.9
  [74087812] Random123 v1.6.0
  [fb686558] RandomExtensions v0.4.3
  [e6cf234a] RandomNumbers v1.5.3
  [c1ae055f] RealDot v0.1.0
  [3cdcf5f2] RecipesBase v1.2.1
  [01d81517] RecipesPipeline v0.6.3
  [731186ca] RecursiveArrayTools v2.32.0
  [f2c3362d] RecursiveFactorization v0.2.12
  [189a3867] Reexport v1.2.2
  [42d2dcc6] Referenceables v0.1.2
  [29dad682] RegularizationTools v0.6.0
⌅ [05181044] RelocatableFolders v0.3.0
  [ae029012] Requires v1.3.0
  [ae5879a3] ResettableStacks v1.1.1
  [37e2e3b7] ReverseDiff v1.14.1
  [79098fc4] Rmath v0.7.0
  [47965b36] RootedTrees v2.13.0
  [7e49a35a] RuntimeGeneratedFunctions v0.5.3
  [3cdde19b] SIMDDualNumbers v0.1.1
  [94e857df] SIMDTypes v0.1.0
  [476501e8] SLEEFPirates v0.6.35
  [0bca4576] SciMLBase v1.53.2
  [1ed8b502] SciMLSensitivity v7.7.0
  [30cb0354] SciMLTutorials v1.0.0
  [6c6a2e73] Scratch v1.1.1
⌅ [efcf1570] Setfield v0.8.2
  [605ecd9f] ShowCases v0.1.0
  [992d4aef] Showoff v1.0.3
  [777ac1f9] SimpleBufferStream v1.1.0
  [699a6c99] SimpleTraits v0.9.4
  [66db9d55] SnoopPrecompile v1.0.1
  [ed01d8cd] Sobol v1.5.0
  [b85f4697] SoftGlobalScope v1.1.0
  [a2af1166] SortingAlgorithms v1.0.1
  [47a9eef4] SparseDiffTools v1.26.2
  [276daf66] SpecialFunctions v2.1.7
  [171d559e] SplittablesBase v0.1.14
  [860ef19b] StableRNGs v1.0.0
  [aedffcd0] Static v0.7.6
  [90137ffa] StaticArrays v1.5.6
  [1e83bf80] StaticArraysCore v1.3.0
  [82ae8749] StatsAPI v1.5.0
  [2913bbd2] StatsBase v0.33.21
  [4c63d2b9] StatsFuns v1.0.1
  [9672c7b4] SteadyStateDiffEq v1.9.0
  [789caeaf] StochasticDiffEq v6.53.0
  [7792a7ef] StrideArraysCore v0.3.15
  [69024149] StringEncodings v0.3.5
  [09ab397b] StructArrays v0.6.12
  [53d494c1] StructIO v0.3.0
  [c3572dad] Sundials v4.10.1
  [d1185830] SymbolicUtils v0.19.11
  [0c5d862f] Symbolics v4.10.4
  [3783bdb8] TableTraits v1.0.1
  [bd369af6] Tables v1.7.0
  [62fd8b95] TensorCore v0.1.1
⌅ [8ea1fca8] TermInterface v0.2.3
  [5d786b92] TerminalLoggers v0.1.6
  [8290d209] ThreadingUtilities v0.5.0
  [ac1d9e8a] ThreadsX v0.1.10
  [a759f4b9] TimerOutputs v0.5.21
  [0796e94c] Tokenize v0.5.24
  [9f7883ad] Tracker v0.2.21
  [3bb67fe8] TranscodingStreams v0.9.9
  [28d57a85] Transducers v0.4.73
  [a2a6695c] TreeViews v0.3.0
  [d5829a12] TriangularSolve v0.1.13
  [410a4b4d] Tricks v0.1.6
  [5c2747f8] URIs v1.4.0
  [3a884ed6] UnPack v1.0.2
  [d9a01c3f] Underscores v3.0.0
  [1cfade01] UnicodeFun v0.4.1
  [1986cc42] Unitful v1.11.0
  [41fe7b60] Unzip v0.2.0
  [3d5dd08c] VectorizationBase v0.21.47
  [81def892] VersionParsing v1.3.0
  [19fa3120] VertexSafeGraphs v0.2.0
⌃ [44d3d7a6] Weave v0.10.9
  [ddb6d928] YAML v0.4.7
  [c2297ded] ZMQ v1.2.1
  [e88e6eb3] Zygote v0.6.47
  [700de1a5] ZygoteRules v0.2.2
  [6e34b625] Bzip2_jll v1.0.8+0
  [83423d85] Cairo_jll v1.16.1+1
  [3bed1096] Cuba_jll v4.2.2+1
  [7bc98958] Cubature_jll v1.0.5+0
  [5ae413db] EarCut_jll v2.2.3+0
⌅ [7cc45869] Enzyme_jll v0.0.33+0
  [2e619515] Expat_jll v2.4.8+0
  [b22a6f82] FFMPEG_jll v4.4.2+0
  [a3f928ae] Fontconfig_jll v2.13.93+0
  [d7e528f0] FreeType2_jll v2.10.4+0
  [559328eb] FriBidi_jll v1.0.10+0
  [0656b61e] GLFW_jll v3.3.8+0
  [d2c73de3] GR_jll v0.66.2+0
  [78b55507] Gettext_jll v0.21.0+0
  [7746bdde] Glib_jll v2.68.3+2
  [3b182d85] Graphite2_jll v1.3.14+0
  [2e76f6c2] HarfBuzz_jll v2.8.1+1
  [aacddb02] JpegTurbo_jll v2.1.2+0
  [c1c5ebd0] LAME_jll v3.100.1+0
  [88015f11] LERC_jll v3.0.0+1
  [dad2f222] LLVMExtra_jll v0.0.16+0
  [dd4b983a] LZO_jll v2.10.1+0
  [e9f186c6] Libffi_jll v3.2.2+1
  [d4300ac3] Libgcrypt_jll v1.8.7+0
  [7e76a0d4] Libglvnd_jll v1.3.0+3
  [7add5ba3] Libgpg_error_jll v1.42.0+0
  [94ce4f54] Libiconv_jll v1.16.1+1
  [4b2f31a3] Libmount_jll v2.35.0+0
  [89763e89] Libtiff_jll v4.4.0+0
  [38a345b3] Libuuid_jll v2.36.0+0
  [e7412a2a] Ogg_jll v1.3.5+1
  [458c3c95] OpenSSL_jll v1.1.17+0
  [efe28fd5] OpenSpecFun_jll v0.5.5+0
  [91d4177d] Opus_jll v1.3.2+0
  [2f80f16e] PCRE_jll v8.44.0+0
  [30392449] Pixman_jll v0.40.1+0
  [ea2cea3b] Qt5Base_jll v5.15.3+1
  [f50d1b31] Rmath_jll v0.3.0+0
  [fb77eaff] Sundials_jll v5.2.1+0
  [a2964d1f] Wayland_jll v1.19.0+0
  [2381bf8a] Wayland_protocols_jll v1.25.0+0
  [02c8fc9c] XML2_jll v2.9.14+0
  [aed1982a] XSLT_jll v1.1.34+0
  [4f6342f7] Xorg_libX11_jll v1.6.9+4
  [0c0b7dd1] Xorg_libXau_jll v1.0.9+4
  [935fb764] Xorg_libXcursor_jll v1.2.0+4
  [a3789734] Xorg_libXdmcp_jll v1.1.3+4
  [1082639a] Xorg_libXext_jll v1.3.4+4
  [d091e8ba] Xorg_libXfixes_jll v5.0.3+4
  [a51aa0fd] Xorg_libXi_jll v1.7.10+4
  [d1454406] Xorg_libXinerama_jll v1.1.4+4
  [ec84b674] Xorg_libXrandr_jll v1.5.2+4
  [ea2f1a96] Xorg_libXrender_jll v0.9.10+4
  [14d82f49] Xorg_libpthread_stubs_jll v0.1.0+3
  [c7cfdc94] Xorg_libxcb_jll v1.13.0+3
  [cc61e674] Xorg_libxkbfile_jll v1.1.0+4
  [12413925] Xorg_xcb_util_image_jll v0.4.0+1
  [2def613f] Xorg_xcb_util_jll v0.4.0+1
  [975044d2] Xorg_xcb_util_keysyms_jll v0.4.0+1
  [0d47668e] Xorg_xcb_util_renderutil_jll v0.3.9+1
  [c22f9ab0] Xorg_xcb_util_wm_jll v0.4.1+1
  [35661453] Xorg_xkbcomp_jll v1.4.2+4
  [33bec58e] Xorg_xkeyboard_config_jll v2.27.0+4
  [c5fb5394] Xorg_xtrans_jll v1.4.0+3
  [8f1865be] ZeroMQ_jll v4.3.4+0
  [3161d3a3] Zstd_jll v1.5.2+0
  [a4ae2306] libaom_jll v3.4.0+0
  [0ac62f75] libass_jll v0.15.1+0
  [f638f0a6] libfdk_aac_jll v2.0.2+0
  [b53b4c65] libpng_jll v1.6.38+0
  [a9144af2] libsodium_jll v1.0.20+0
  [f27f6e37] libvorbis_jll v1.3.7+1
  [1270edf5] x264_jll v2021.5.5+0
  [dfaa095f] x265_jll v3.5.0+0
  [d8fb68d0] xkbcommon_jll v1.4.1+0
  [0dad84c5] ArgTools v1.1.1
  [56f22d72] Artifacts
  [2a0f44e3] Base64
  [ade2ca70] Dates
  [8bb1440f] DelimitedFiles
  [8ba89e20] Distributed
  [f43a241f] Downloads v1.6.0
  [7b1f6079] FileWatching
  [9fa8497b] Future
  [b77e0a4c] InteractiveUtils
  [4af54fe1] LazyArtifacts
  [b27032c2] LibCURL v0.6.3
  [76f85450] LibGit2
  [8f399da3] Libdl
  [37e2e46d] LinearAlgebra
  [56ddb016] Logging
  [d6f4376e] Markdown
  [a63ad114] Mmap
  [ca575930] NetworkOptions v1.2.0
  [44cfe95a] Pkg v1.8.0
  [de0858da] Printf
  [3fa0cd96] REPL
  [9a3f8284] Random
  [ea8e919c] SHA v0.7.0
  [9e88b42a] Serialization
  [1a1011a3] SharedArrays
  [6462fe0b] Sockets
  [2f01184e] SparseArrays
  [10745b16] Statistics
  [4607b0f0] SuiteSparse
  [fa267f1f] TOML v1.0.0
  [a4e569a6] Tar v1.10.0
  [8dfed614] Test
  [cf7118a7] UUIDs
  [4ec0a83e] Unicode
  [e66e0078] CompilerSupportLibraries_jll v0.5.2+0
  [deac9b47] LibCURL_jll v7.84.0+0
  [29816b5a] LibSSH2_jll v1.10.2+0
  [c8ffd9c3] MbedTLS_jll v2.28.0+0
  [14a3606d] MozillaCACerts_jll v2022.2.1
  [4536629a] OpenBLAS_jll v0.3.20+0
  [05823500] OpenLibm_jll v0.8.1+0
  [bea87d4a] SuiteSparse_jll v5.10.1+0
  [83775a58] Zlib_jll v1.2.12+3
  [8e850b90] libblastrampoline_jll v5.1.1+0
  [8e850ede] nghttp2_jll v1.48.0+0
  [3f19e933] p7zip_jll v17.4.0+0
Info Packages marked with ⌃ and ⌅ have new versions available, but those with ⌅ cannot be upgraded. To see why use `status --outdated -m`
```
