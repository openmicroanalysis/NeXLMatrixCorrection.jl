# NeXLMatrixCorrection

The matrix correction package in the NeXL microanalysis library for Julia.  `NeXLMatrixCorrection` depends upon
`NeXLUncertainties` and `NeXLCore`.

`NeXLMatrixCorrection` and some of its dependences are not yet available in the Julia registry so they must be installed using the URL.

```julia
using Pkg;
Pkg.add.([
  "https://github.com/usnistgov/BoteSalvatICX.jl.git",
  "https://github.com/usnistgov/FFAST.jl.git",
  "https://github.com/NicholasWMRitchie/NeXLUncertainties.jl.git",
  "https://github.com/NicholasWMRitchie/NeXLCore.jl.git",
  "https://github.com/NicholasWMRitchie/NeXLMatrixCorrection.jl.git",
  "https://github.com/NicholasWMRitchie/NeXLSpectrum.jl.git"  # Optional
])
```

Currently `NeXLMatrixCorrection` implements the XPP matrix correction and Reed fluorescence correction algorithms for bulk and coated samples.  The library is designed to make it easy to add additional algorithms.

Primarily the algorithms in `NeXLMatrixCorrection` are designed to take a `Vector{NeXLCore.KRatio}` and return a `NeXLCore.Material`.  Since they are intended for both WDS and EDS, the k-ratio can represent one or more characteristic X-ray lines from a single element.  K-ratios compare a measured intensity with the intensity from a reference (standard) material. Typically, these two materials are measured at the same beam energy but multiple beam energy measurements are also supported.

The primary methods are
```julia
quantify(  # Generic method for EDS, WDS or mixed data
  sample::Union{String, Label}, # Sample name or Label
  measured::Vector{KRatio};     # The k-ratios
  mc::Type{<:MatrixCorrection}=XPP,  # Default algorithm choices
  fc::Type{<:FluorescenceCorrection}=ReedFluorescence,
  cc::Type{<:CoatingCorrection}=Coating)::IterationResult
quantify(ffr::FilterFitResult)::IterationResult  # Specialized for the results from fitted EDS spectra

# where

KRatio(
    lines::AbstractVector{CharXRay},  # CharXRay or X-rays measured
    unkProps::Dict{Symbol,<:Any},     # Properties of the measurement ( :BeamEnery, :TakeOffAngle )
    stdProps::Dict{Symbol,<:Any},     # Properties of the standard ( :BeamEnery, :TakeOffAngle )
    standard::Material,               # Composition of the standard
    kratio::AbstractFloat,            # The k-ratio (can be an UncertainValue)
)
```

### An example
```julia
julia> lbl = label("K458")
julia> unkProps = Dict(:BeamEnergy=>15.0e3, :TakeOffAngle=>deg2rad(40.0))
julia> stdProps = unkProps # Same for both (in this case...)
julia> krs = [
    KRatio([n"O K-L3"], unkProps, stdProps, mat"SiO2", 0.746227 ),
    KRatio([n"Si K-L3"], unkProps, stdProps, mat"SiO2", 0.441263 ),
    KRatio([n"Zn K-L3"], unkProps, stdProps, mat"Zn", 0.027776 ),
    KRatio([n"Ba L3-M5"], unkProps, stdProps, mat"BaCl", 0.447794 )
]
julia> res = quantify(lbl, krs)
```
Converged to K458[Si=0.2311,Ba=0.4212,O=0.3192,Zn=0.0307] in 7 steps
