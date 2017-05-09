# erd
N. Flocke and V. Lotrich's electron repulsion integrals from AcesIII wrapped in CMake for Psi4 (https://github.com/psi4/psi4)

### History

This is the ERD project by N. Flocke and V. Lotrich of QTP
of University of Florida.

ERD is written in Fortran. Its best resource is 
[doi 10.1002/jcc.21018](http://onlinelibrary.wiley.com/doi/10.1002/jcc.21018/abstract).

### This Repository

ERD has been in the *ab initio* quantum chemistry package Psi4
(http://psicode.org/, https://github.com/psi4/psi4) since about 2014. In Psi4,
it builds with `cmake` and has an interface to C++ and Psi4 libmints internals designed
by @andysim and @bennybp. Manual for ERD+Psi4 at http://psicode.org/psi4manual/master/erd.html.
This repository is ERD wrapped up nicely in CMake.

#### Version

This codebase was copied from upstream and a few adaptations have been applied
to the integral normalization to make it work with Psi4. These adaptations have
been notated with "psi4" in the code.The erd package doesn't seem to have its
own versioning, so the version number of AcesIII was adopted here.

#### License

From headers in the code, the applicable license is
the GNU General Public License, version 2 or later
([GPL-2.0+](https://opensource.org/licenses/GPL-2.0+)).

```
ERD - a library for efficient electronic integrals and their
generalized derivatives
Copyright (c) 2003-2010 University of Florida

This program is free software; you can redistribute it and/or
modify it under the terms of the GNU General Public License
as published by the Free Software Foundation; either version 2
of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.
```

#### Building

```bash
cmake -H. -Bobjdir
cd objdir && make
make install
```

The build is also responsive to

* static/shared toggle `BUILD_SHARED_LIBS`
* install location `CMAKE_INSTALL_PREFIX`
* of course, `CMAKE_Fortran_COMPILER`, `CMAKE_C_COMPILER`, and `CMAKE_Fortran_FLAGS`

See [CMakeLists.txt](CMakeLists.txt) for options details. All these build options should be passed as `cmake -DOPTION`.

#### Detecting

This project installs with `erdConfig.cmake`, `erdConfigVersion.cmake`, and `erdTargets.cmake` files suitable for use with CMake [`find_package()`](https://cmake.org/cmake/help/v3.2/command/find_package.html) in `CONFIG` mode.

* `find_package(erd)` - find any erd libraries and headers
* `find_package(erd 3.0.6 EXACT CONFIG REQUIRED COMPONENTS static)` - find erd exactly version 3.0.6 built with static libraries or die trying

See [erdConfig.cmake.in](cmake/erdConfig.cmake.in) for details of how to detect the Config file and what CMake variables and targets are exported to your project.

#### Using

After `find_package(erd ...)`,

* test if package found with `if(${erd_FOUND})` or `if(TARGET erd::erd)`
* link to library (establishes dependency), including header and definitions configuration with `target_link_libraries(mytarget erd::erd)`
* include header files using `target_include_directories(mytarget PRIVATE $<TARGET_PROPERTY:erd::erd,INTERFACE_INCLUDE_DIRECTORIES>)`
* compile target applying `-DUSING_erd` definition using `target_compile_definitions(mytarget PRIVATE $<TARGET_PROPERTY:erd::erd,INTERFACE_COMPILE_DEFINITIONS>)`
