# Changelog

All notable changes to this E3SM_tai working copy are documented here.
Differentiates **Features**, **Fixed**, and **Breaking changes**.

## [Unreleased]

### Features
- **Add `pflogin` machine support** (ORNL CADES baseline cluster: `pflogin*` login /
  `blc*` compute nodes, Rocky Linux 9.7, slurm, account/qos/partition `hpcl-cli185`).
  - `cime_config/machines/config_machines.xml`: new `<machine MACH="pflogin">` —
    gnu/openmpi toolchain via Lmod (gcc 12.4.0, openmpi 5.0.5, hdf5/netcdf/pnetcdf,
    cmake, miniforge3); scratch `/scratch/$USER/e3sm_scratch`; inputdata
    `/projects/hpcl-cli185/world-shared/e3sm/inputdata`. Env vars wire the prebuilt
    PETSc 3.21.6 and alquimia/PFLOTRAN "v2021" (`PETSC_DIR`, `ALQUIMIA_PATH`,
    `PFLOTRAN_SRC`), and prepend a conda perl (with `XML::LibXML`) to PATH for
    ELM build-namelist/configure.
  - `cime_config/machines/cmake_macros/gnu_pflogin.cmake`: new compiler/link macro
    (netcdf/hdf5/pnetcdf/openblas SLIBS; `-fallow-argument-mismatch -fno-range-check`;
    lustre PIO hints). Alquimia/PFLOTRAN/PETSc linking handled by base `gnu.cmake`.
  - `cime_config/machines/config_batch.xml`: new `pflogin` slurm batch entry.

### Fixed
- **ELM alquimia coupling compile + unit fix**
  (`components/elm/src/external_models/emi/src/em/alquimia/ExternalModelAlquimiaMod.F90`):
  - `run_column_onestep` was called with two lateral-flux actual arguments
    (`qflx_drain_l2e/dt` and `qflx_lat_aqu_l2e`) while the subroutine declares one
    (`lat_flow`) → "More actual than formal arguments" build failure. Collapsed the
    call to a single `qflx_lat_aqu_l2e(c,:)/dt`.
  - Added missing `*dt` to the `qflx_lat_aqu_l2e` mass-balance correction so it is unit
    consistent (vertical advective flux is a rate, mm/s; `qflx_lat_aqu_l2e` is
    integrated over the step, mm). Both changes match the working f9y reference copy.
