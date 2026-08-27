# UUV top-fin overset case (OpenFOAM v2212)

This is a three-dimensional incompressible-water case for a UUV aligned with
NED axes: +X forward and +Z down. The imposed relative flow is 4 kn in -X:

`U = (-2.05777778 0 0) m/s`

The fixed background mesh is cut around `body.stl`. A separate moving mesh is
cut inside `finOversetDomain.stl` and around `fin.stl`; the outer STL is emitted
as the `overset` patch and the component is assigned to cell zone `finZone`.
For meshing only, `Allmesh` translates a copy of the supplied overset cylinder
25 mm in +Z. Its original end plane coincides with the fin/body root and creates
an open interpolation patch; the small penetration closes that interface and
provides robust overlap inside the solid body.

## Solvers

The initialization uses `overSimpleFoam`. OpenFOAM.com v2212 does not ship an
executable named `overPimpleFoam`; its supported transient dynamic-overset
solver is `overPimpleDyMFoam`, which this case uses.

## Motion

The fin and its overset mesh rotate about the NED Z axis through
`(-1.2 0 -0.1)`. The tabulated yaw rises linearly from 0 to +5 degrees at
35 rpm (210 deg/s), reaching 5 degrees at 0.023809524 s, and then holds.
Reverse the signs of the yaw values in `constant/finMotion.dat` for the
opposite actuation direction.

## Run

Load the v2212 environment, then run:

```sh
./Allmesh
./AllrunSteady
./AllrunTransient
```

`AllrunTransient` copies the latest steady fields back to time 0 and removes
the old positive iteration directories before starting physical time. Keep a
copy of the steady time directory first if you need it independently.

The default mesh is deliberately moderate for a runnable baseline. Perform a
mesh-convergence and near-wall-resolution study before treating forces as
engineering results. Water properties are `nu=1e-6 m2/s`, `rho=1025 kg/m3`,
with k-omega SST turbulence and nominal 5% inlet turbulence intensity.

The standard `checkMesh` result is `Mesh OK` (maximum non-orthogonality about
48 degrees and maximum skewness about 0.53). The additional strict check in
`log.checkMesh.full` flags concave snappyHexMesh polyhedra near curved/snap
transitions; review that report and tighten the production mesh as part of the
mesh-convergence study.
