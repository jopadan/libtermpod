cmake_minimum_required(VERSION 3.0)
project(termpod LANGUAGES C)

set(PROJECT_VERSION 1.0)
set(PROJECT_DESCRIPTION "POD game archive format library written in C for all existing versions")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

if(CMAKE_C_COMPILER_LOADED)
  include (CheckIncludeFile)
  include (CheckCSourceCompiles)
elseif(CMAKE_CXX_COMPILER_LOADED)
  include (CheckIncludeFileCXX)
  include (CheckCXXSourceCompiles)
else()
  message(FATAL_ERROR "FindMath only works if either C or CXX language is enabled")
endif()

include(GNUInstallDirs)
include(FindPkgConfig)

if(MSYS AND CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
set(CMAKE_INSTALL_PREFIX $ENV{MINGW_PREFIX} CACHE PATH $ENV{MINGW_PREFIX} FORCE)
endif()

if(WIN32)
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
else()
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
endif()

#set( CMAKE_C_STANDARD_REQUIRED ON )
#set( CMAKE_C_STANDARD gnu2x )

set(SOURCE_FILES
	src/path.c
	src/mkdir_p.c
	src/pod_crc.c
	src/pod_common.c
	src/termpod.c
	src/pod_zip.c
	src/pod1.c
	src/pod2.c
	src/pod3.c
	src/pod4.c
	src/pod5.c
	src/pod6.c
	src/epd.c
)

set(INCLUDE_FILES
	src/path.h
	src/mkdir_p.h
	src/pod_crc.h
	src/pod_common.h
	src/termpod.h
	src/pod_zip.h
	src/pod1.h
	src/pod2.h
	src/pod3.h
	src/pod4.h
	src/pod5.h
	src/pod6.h
	src/epd.h
)

set(EXAMPLE_SOURCE_FILES
)

pkg_search_module(ZIP REQUIRED libzip)
#pkg_search_module(CRC REQUIRED libcrcle)

# termpod library
add_library( ${PROJECT_NAME} SHARED ${SOURCE_FILES} ${INCLUDE_FILES}
	${CRC_INCLUDE} ${ZIP_INCLUDE})
target_compile_options( ${PROJECT_NAME} PUBLIC ${CRC_CFLAGS} ${ZIP_CFLAGS})
target_include_directories(${PROJECT_NAME} PUBLIC ${CRC_INCLUDEDIR} ${ZIP_INCLUDEDIR}
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
			$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
			)
target_link_libraries( ${PROJECT_NAME} PUBLIC ${CRC_LIBRARIES} ${ZIP_LIBRARIES} )


target_include_directories(${PROJECT_NAME} PUBLIC ${CRC_INCLUDEDIR} ${ZIP_INCLUDEDIR}
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
			$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
			)
set(install_prefix "${CMAKE_INSTALL_PREFIX}")
set(install_libdir "${CMAKE_INSTALL_LIBDIR}")
set(install_includedir "${CMAKE_INSTALL_INCLUDEDIR}")
set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(project_config "${PROJECT_NAME}Config.cmake")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in"
	"${project_config}"
	INSTALL_DESTINATION "${config_install_dir}"
)

configure_file(lib${PROJECT_NAME}.pc.in ${PROJECT_BINARY_DIR}/lib${PROJECT_NAME}.pc @ONLY)

set(ENV{PKG_CONFIG_PATH} "${PROJECT_BINARY_DIR}:$ENV{PKG_CONFIG_PATH}")

install(TARGETS ${PROJECT_NAME} EXPORT TermPODExport
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	PUBLIC_HEADER DESTINATION
	${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
	)

install(EXPORT TermPODExport
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/TermPOD"
	FILE ${project_config}
	NAMESPACE TermPOD::
)

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindTermPOD.cmake DESTINATION
	${CMAKE_INSTALL_LIBDIR}/cmake/TermPOD)

install(FILES ${INCLUDE_FILES} DESTINATION
	${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/)

install(FILES ${PROJECT_BINARY_DIR}/lib${PROJECT_NAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/)
