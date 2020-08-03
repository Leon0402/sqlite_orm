cmake_minimum_required (VERSION 3.16.0)

# PACKAGE_VERSION is used by cpack scripts currently
# Both sqlite_orm_VERSION and PACKAGE_VERSION should be the same for now

set(sqlite_orm_VERSION "1.3.0")
set(PACKAGE_VERSION ${sqlite_orm_VERSION})

project("sqlite_orm" VERSION ${PACKAGE_VERSION})


set(CMAKE_VERBOSE_MAKEFILE ON)

message(STATUS "Configuring ${CMAKE_PROJECT_NAME} ${sqlite_orm_VERSION}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

option(SQLITE_ORM_ENABLE_CXX17 "Enable C++17 features" OFF)

include(CTest)

### Dependencies
add_subdirectory(dependencies)

### Main Build Targets
set(SqliteOrm_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/include")
add_library(sqlite_orm INTERFACE)
add_library(sqlite_orm::sqlite_orm ALIAS sqlite_orm)

find_package(SQLite3 REQUIRED)
target_link_libraries(sqlite_orm INTERFACE SQLite::SQLite3)

target_compile_features(sqlite_orm INTERFACE cxx_std_14)
target_compile_definitions(sqlite_orm INTERFACE SQLITE_ORM_HAS_CXX17=${SQLITE_ORM_ENABLE_CXX17})

target_sources(sqlite_orm INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include/sqlite_orm/sqlite_orm.h>)

target_include_directories(sqlite_orm INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>)

include(ucm)

if (MSVC)
	string(REGEX REPLACE "/RTC(su|[1su])" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")	
	string(REGEX REPLACE "/RTC(su|[1su])" "" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")	
	string(REGEX REPLACE "/RTC(su|[1su])" "" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")	
	add_compile_options(/EHsc)
	add_compile_options(/Zc:__cplusplus)

	if ("${CMAKE_GENERATOR}" MATCHES "(Win64|x64)")
		message(STATUS "Add /bigobj flag to compiler")
    	add_compile_options(/bigobj)
	endif()
endif()

ucm_print_flags()
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")

# Tests
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    add_subdirectory(tests)
endif()

option(BUILD_EXAMPLES ON)
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_EXAMPLES)
	add_subdirectory(examples)
endif()

### Packaging
add_subdirectory(packaging)
