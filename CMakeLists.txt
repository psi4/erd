cmake_minimum_required(VERSION 3.0)
project(liberd LANGUAGES Fortran C)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

################################### Options ####################################
include(Psi4Macros)
option_with_default(CMAKE_BUILD_TYPE "Build type" Release)
option_with_print(BUILD_SHARED_LIBS "Build final library as shared, not static" OFF)
option_with_default(BUILD_FPIC "Libraries will be compiled with position independent code" ON)
if(${BUILD_SHARED_LIBS} AND NOT ${BUILD_FPIC})
    message(FATAL_ERROR "BUILD_SHARED_LIBS ON and BUILD_FPIC OFF are incompatible, as shared library requires position independent code")
endif()
option_with_print(ENABLE_GENERIC "Enable mostly static linking in shared library" OFF)
option_with_flags(ENABLE_XHOST "Enable processor-specific optimization" ON
                    "-xHost" "-march=native")
# CODE_COVERAGE should be tested and probably propagated to external projects
#option_with_flags(ENABLE_CODE_COVERAGE "Enables details on code coverage" OFF
#                    "-ftest-coverage")

######################### Process & Validate Options ###########################
include(autocmake_safeguards)
include(autocmake_static_library)

################################# Main Project #################################
include(GNUInstallDirs)
include(FortranCInterface)
include(CMakePackageConfigHelpers)

set(PN ${PROJECT_NAME})

# <<<  Build  >>>

FortranCInterface_HEADER(ERD_MANGLE.h
                         MACRO_NAMESPACE "ERD_MANGLE_"
                         SYMBOL_NAMESPACE "ERD_MANGLE_")
set(raw_sources_list gener_eri_derv_batch.f derv_int2d_to_e0cd.f memory_eri_batch.f
                     e0f0_pcgto_block.f cartesian_norms.f hrr_transform.f csgto.f
                     ssss_pcgto_block.f sspp_pcgto_block.f set_abcd.f memory_eri_derv_batch.f
                     ctr_1st_half.f 1111_csgto.f normalize_cartesian.f ctr_4index_block.f
                     dsqmin_line_segments.f rys_4_roots_weights.f derv_int2d_to_0000.f
                     derv_def_blocks.f ctr_2nd_half_new.f rys_roots_weights.f
                     derv_int2d_to_e000.f transpose_batch.f print_batch.f
                     rys_1_roots_weights.f derv_csgto.f int2d_to_e000.f set_derv_abcd.f
                     memory_1111_csgto.f move_ry.f hrr_matrix.f ctr_4index_reorder.f
                     memory_hrr_correction.f derv_int2d_to_a000.f 2d_coefficients.f
                     derv_int2d_to_abc0.f sppp_pcgto_block.f ctr_2nd_half_update.f
                     derv_pcgto_block.f hrr_step.f 2d_atom_coefficients.f gener_eri_batch.f
                     rys_3_roots_weights.f 2d_derv_pcd_integrals.f xyz_to_ry_abcd.f
                     2d_derv_abcd_integrals.f set_ij_kl_pairs.f rys_2_roots_weights.f
                     derv_int2d_to_ab00.f e0f0_def_blocks.f 2d_pq_integrals.f
                     atom_int2d_to_e0f0.f derv_int2d_to_abcd.f derv_int2d_to_e0c0.f
                     1111_def_blocks.f set_derv_sequence.f map_ijkl_to_ikjl.f ctr_rs_expand.f
                     2d_abcd_integrals.f xyz_to_ry_matrix.f sssp_pcgto_block.f
                     atom_int2d_to_e000.f 2d_pcd_integrals.f ctr_tu_expand.f
                     memory_derv_csgto.f spherical_transform.f memory_csgto.f int2d_to_e0f0.f
                     rys_5_roots_weights.f pppp_pcgto_block.f 2d_atom_pq_integrals.f
                     rys_x_roots_weights.f prepare_ctr.F)

set(src_prefix "src/erd__")
string(REGEX REPLACE "([^;]+)" "${src_prefix}\\1" sources_list "${raw_sources_list}")

# STATIC/SHARED on below governed by BUILD_SHARED_LIBS
add_library(erd ${sources_list})
set_target_properties(erd PROPERTIES POSITION_INDEPENDENT_CODE ${BUILD_FPIC})
if(${BUILD_SHARED_LIBS})
    target_link_libraries(erd PRIVATE ${LIBC_INTERJECT})
endif()

# <<<  Install  >>>

install(FILES ${CMAKE_BINARY_DIR}/ERD_MANGLE.h
        DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}/ERD)
install(TARGETS erd
        EXPORT "${PN}Targets"
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

# <<<  Export Interface  >>>

target_compile_definitions(erd INTERFACE USING_${PN})
target_include_directories(erd INTERFACE
                           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

# <<<  Export Config  >>>

set(CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/cmake/${PN}")
configure_package_config_file(${PN}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake"
                              INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
                                 VERSION 9.9.9
                                 COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(EXPORT "${PN}Targets"
        NAMESPACE "${PN}::"
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})

