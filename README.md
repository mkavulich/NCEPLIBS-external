# NCEPLIBS-external

## Warning

This documentation as well as the libraries are work in progress and currently undergoing rapid development. Do not expect any of this to work.

## Introduction

This is a compilation of third-party libraries required to build NCEPLIBS and by extension the UFS weather model. 

It inclues the following libraries:

1. MPI: openmpi-4.0.2
2. zlib: zlib-1.2.11
3. HDF5: hdf5-1.10.4
4. NetCDF: netcdf-c-4.7.3, netcdf-fortran-4.5.2
5. libpng: libpng-1.6.35
6. libjpeg: jpeg-9.1
7. Jasper: jasper-2.0.16
8. WGRIB2: wgrib-2.0.8
9. ESMF: esmf-8.0.0
10. (optional) CMake: cmake-3.16.3

## Building

```
git clone https://github.com/NOAA-EMC/NCEPLIBS-external
cd NCEPLIBS-external
git submodule update --init --recursive
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=INSERT_PATH_HERE ..
make <-jx>
```
If `-DCMAKE_INSTALL_PREFIX=` is omitted, the libraries will be installed in directory `install` underneath the `build` directory. The optional argument `<-jx>` speeds up the build process by using `x` parallel compile tasks.

By default, NCEPLIBS-external will build and install all libraries listed above except the optional CMake (see next section when this is needed). This is primarily targeted for users who are setting up their own workstations in order to get a consistent software stack. It is important that the MPI wrappers use the same compiler that is used to compile the other external libraries, the NCEP libraries and any UFS application that depends on those. 

Users working on HPC systems should use the MPI libraries installed by the system administrators for the compiler they want to use. Usually compilers and MPI libraries can be loaded as modules on those systems. If other dependencies such as netCDF are also provided as modules for the target compiler, then these should also be used. In this case, users need to turn off explicitly the build of those libraries as follows:

*How to turn off building ...*
1. MPI: Add `-DBUILD_MPI=OFF` to the cmake flags. If CMake cannot find the MPI library, make sure that the wrappers `mpicc` and `mpif90` are in the `PATH` environment variable, and set the environment variable `MPI_ROOT` to the directory where MPI was installed.
2. zlib: zlib will be built as dependency for NetCDF or libpng. If neither of those is built, the zlib build is also turned off.
3. HDF5: will be built as dependency for NetCDF.  If netCDF is not built, the HDF5 build is also turned off.
4. NetCDF: Add `-DBUILD_NETCDF=OFF` to the cmake flags. If CMake cannot find the netCDF/netCDF Fortran library, set the environment variable `NETCDF` to the directory where NetCDF was installed. On some systems, the netCDF-c and netCDF-fortran libraries are installed in separate locations. In this case, set set the environment variable `NETCDF` to the directory where netcdf-c was installed, and `NETCDF_FORTRAN` to the directory where netcdf-fortran was installed.
5. libpng: Add `-DBUILD_LIBPNG=OFF` to the cmake flags. If CMake cannot find the libpng library, set the environment variable `LIBPNG_ROOT` to the directory where libpng was installed.
6. libjpeg: libjpeg will be built as dependency for Jasper. If Jasper is not built, the libjpeg build is also turned off.
7. Jasper: Add `-DBUILD_JASPER=OFF` to the cmake flags. If CMake cannot find the Jasper library, set the environment variable `JASPER_ROOT` to the directory where Jasper was installed.
8. WGRIB2: Add `-DBUILD_WGRIB2=OFF` to the cmake flags. If CMake cannot find the WGRIB2 Fortran modules or library, set the environment variable `WGRIB2_ROOT` to the directory where WGRIB2 was installed. Note that WGRIB2 installations vary between systems. For the NCEPLIBS build to work, the WGRIB2 Fortran modules are expected in `WGRIB2_ROOT/include` and the WGRIB2 library in `WGRIB2_ROOT/lib`.
9. ESMF: Add `-DBUILD_ESMF=OFF` to the cmake flags and set the environment variable `ESMFMKFILE` to point to the file `esmf.mk` of the ESMF installation.

The above options to turn off selected components can also be used by advanced users who already have parts of the software stack installed for the UFS.

## Requirements

1. CMake version 3.12 or newer. If the `cmake` command above fails because the existing CMake version is too old, you need to install a newer version. The NCEPLIBS-external code contains
`cmake-3.16.3` in subdirectory `cmake-src`. To install this version:
```
cd ../cmake-src # assuming you are still in the build directory
./bootstrap --prefix=INSERT_PATH_HERE
make
make install
cd ../build
rm -fr CMakeCache.txt CMakeFiles
```
For building NCEPLIBS-external, use `/full_path_to_where_you_installed_cmake/bin/cmake` instead of just `cmake`.

2. A supported C/C++ and Fortran compiler, see table below. Other versions may work, in particular if close to the versions listed below. If the chosen compiler is not the default compiler on the system, set the environment variables `export CC=...`, `export CXX=...`, `export FC=...`, before invoking `cmake`.

| Compiler vendor | Supported (tested) versions                                |
|-----------------|------------------------------------------------------------|
| Intel           | 18.0.3.222, 18.0.5.274, 19.0.2.187, 19.0.5.281             |
| GNU             | 8.3.0, 9.1.0, 9.2.0                                        |

3. A supported MPI library unless installed as part of NCEPLIBS-external, see table below. Other versions may work, in particular if close to the versions listed below. It is recommended to compile the MPI library with the same compilers used to compile NCEPLIBS-external.

| MPI library     | Supported (tested) versions                                |
|-----------------|------------------------------------------------------------|
| MPICH           | 3.3.1, 3.3.2                                               |
| MVAPICH2        | 2.3.3                                                      |
| Open MPI        | 4.0.2                                                      |
| Intel MPI       | 2018.0.4, 2019.6.154                                       |
| SGI MPT         | 2.19                                                       |

## Troubleshooting

1. The cmake step reports an error that it cannot detect the MPI type. Only the MPI libraries listed above have been preconfigured for ESMF. Other libraries can be used by setting `-DMPITYPE=...` in the cmake call, provided that ESMF supports this library (currently supported by ESMF: mpi, mpt, mpich, mpich2, mpich3, mvapich2, intelmpi, lam - see ESMF documentation for details).

2. The build fails because of undefined symbols in any of the libraries. The most likely reason for this error is that cmake found a different version of the library on the system, for example installed by the Linux package manager or homebrew on macOS. While there is no single solution for this problem, the following options have been used successfully:
    - Remove the existing library, if possible (often it is not, because other software on the system depends on it)
    - Take the existing library out of the `PATH` and `LD_LIBRARY_PATH`, if possible (not always, because other software installed in the same location, may be needed). On HPC systems, this often means unloading a specific module.
    - Look inside NCEPLIBS-external's top-level `CMakeLists.txt` if the offending library is one of the build targets, and if yes, turn it off.
    - If the offending library is a prerequisite for a build target (i.e. HDF5 is a prerequisite for NetCDF) and the build target itself is already installed on the system, try to turn off the build target (e.g. `option(BUILD_NETCDF "Build NetCDF?" OFF)`)
    - If the offending library is a prerequisite for a build target (i.e. HDF5 is a prerequisite for NetCDF) and the build target itself is _not_ installed on the system, you need to install the build target using the same method you used for installing the offending library and turn it off in `CMakeLists.txt`.

## Support

MISSING

## Disclaimer

The United States Department of Commerce (DOC) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. DOC has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any claims against the Department of Commerce stemming from the use of its GitHub project will be governed by all applicable Federal law. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by the Department of Commerce. The Department of Commerce seal and logo, or the seal and logo of a DOC bureau, shall not be used in any manner to imply endorsement of any commercial product or activity by DOC or the United States Government.

