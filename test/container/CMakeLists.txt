################################################################################
#   Copyright (C) 2020 GSI Helmholtzzentrum fuer Schwerionenforschung GmbH     #
#                                                                              #
#              This software is distributed under the terms of the             #
#              GNU Lesser General Public Licence (LGPL) version 3,             #
#                  copied verbatim in the file "LICENSE"                       #
################################################################################

add_subdirectory(legacy)


function(fs_spack_container)
  cmake_parse_arguments(ARGS "" "OS;VERSION" "" ${ARGN})

  set(container "${ARGS_OS}.${ARGS_VERSION}")
  set(def "${container}.def")
  set(log "${container}.log")
  set(target "${container}.sif")

  add_custom_command(OUTPUT ${target}
    COMMAND ${CMAKE_COMMAND} -E env ${BASH} -c
    "${SINGULARITY} build -f -F ${target} ${CMAKE_CURRENT_SOURCE_DIR}/${def} > ${log} 2>&1"
    VERBATIM
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Building spack container ${target}, logging to ${CMAKE_CURRENT_BINARY_DIR}/${log}"
  )

  list(APPEND spack_containers ${target})
  set(spack_containers ${spack_containers} PARENT_SCOPE)
endfunction()


fs_spack_container(OS centos VERSION 7)
configure_file(gsi-debian-8-gcc.yaml gsi-debian-8-gcc.yaml COPYONLY) # TODO Make this nicer
fs_spack_container(OS gsi-debian VERSION 8)
fs_spack_container(OS debian VERSION 10)
fs_spack_container(OS fedora VERSION 33)
fs_spack_container(OS opensuse VERSION 15.2)
fs_spack_container(OS ubuntu VERSION 20.04)


add_custom_target(all-spack-containers DEPENDS ${spack_containers})
add_custom_target(all-containers DEPENDS all-spack-containers all-legacy-containers)
