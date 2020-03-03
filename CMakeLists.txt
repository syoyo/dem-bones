project(dem-bones)

cmake_minimum_required(VERSION 3.15)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(CMAKE_CONFIGURATION_TYPES "Release;" CACHE STRING "" FORCE)

# options
option(DEM_BONES_WITH_FBX "Enable FBX support." ON)
option(DEM_BONES_WITH_ALEMBIC "Enable Alembic support." ON)
option(DEM_BONES_USE_SANITIZER "Enable Address Sanitizer." OFF)

# Sanitizer
if(DEM_BONES_USE_SANITIZER)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
    message(STATUS "Address/Memory sanitizer: ENABLED")
else()
    message(STATUS "Address/Memory sanitizer: DISABLED")
endif()

include_directories("${PROJECT_SOURCE_DIR}/include")
find_package(Eigen REQUIRED)
include_directories("${EIGEN_INCLUDE_DIR}")

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}" CACHE PATH "..." FORCE)
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
	set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

file(GLOB CMD_SOURCE
	"src/command/*.h"
	"src/command/*.cpp"
)
add_executable(DemBonesCmd "${CMD_SOURCE}")
set_target_properties(DemBonesCmd PROPERTIES OUTPUT_NAME "DemBones")

find_package(TCLAP)

if (DEM_BONES_WITH_ALEMBIC)
  add_definitions(-DDEM_BONES_ENABLE_ALEMBIC)
  find_package(Alembic)
endif (DEM_BONES_WITH_ALEMBIC)

if (DEM_BONES_WITH_FBX)
  add_definitions(-DDEM_BONES_ENABLE_FBXSDK)
  find_package(FBXSDK)
endif (DEM_BONES_WITH_FBX)



target_include_directories(DemBonesCmd PUBLIC
	"${TCLAP_INCLUDE_DIR}"
	"${FBXSDK_INCLUDE_DIR}"
	"${ALEMBIC_INCLUDE_DIR}"
	"${ALEMBIC_INCLUDE_DIR}/AlembicPrivate/ilmbase"
)

target_link_libraries(DemBonesCmd PUBLIC
	"${FBXSDK_LIBS}"
	"${ALEMBIC_LIBS}"
	"Half"
	"Iex"
	"${ZLIB_LIBS}"
)

if(UNIX)
  if (DEM_BONES_WITH_ALEMBIC)
    # ilmbase component is required
    # Assume ilmbase is installed in the system(e.g. by apt install in Ubunt)
    target_include_directories(DemBonesCmd PUBLIC
          "/usr/include/OpenEXR"
    )

    target_link_libraries(DemBonesCmd PUBLIC
          "Half"
          "Iex"
          ${CMAKE_DL_LIBS}
    )
  endif()

endif()

if(WIN32)
	find_package(Zlib)
	target_link_libraries(DemBonesCmd PUBLIC "${ZLIB_LIBS}")
endif()

if(APPLE)
	set_target_properties(DemBonesCmd PROPERTIES LINK_FLAGS "-Wl,-F/Library/Frameworks")
	target_link_libraries(DemBonesCmd PUBLIC "-framework CoreFoundation")
	set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
	set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
endif()

install(TARGETS DemBonesCmd)
