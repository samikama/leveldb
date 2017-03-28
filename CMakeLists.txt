cmake_minimum_required (VERSION 3.2)
project(LevelDB)
set(CMAKE_CXX_STANDARD 11)
 
function(dbgmsg VAR)
  message(STATUS "Value of ${VAR}=${${VAR}}")
endfunction()
macro(add_lib LIBNAME SHRON STATON )
  dbgmsg("${LIBNAME}")
  dbgmsg("${SHRON}")
  dbgmsg("${STATON}")
  dbgmsg("ARGN")

  if(${SHRON})
    add_library(${LIBNAME} SHARED ${ARGN})
#    target_include_directories(${LIBNAME} PUBLIC ${CMAKE_SOURCE_DIR})
    target_link_libraries(${LIBNAME} Threads::Threads)
  endif()
  if(${STATON})
    add_library(${LIBNAME}_static STATIC ${ARGN})
    set_target_properties(${LIBNAME}_static PROPERTIES OUTPUT_NAME ${LIBNAME})
#    target_include_directories(${LIBNAME}_static PUBLIC ${CMAKE_SOURCE_DIR})
    target_link_libraries(${LIBNAME}_static Threads::Threads)
  endif()
endmacro()
OPTION(BUILD_SHARED "Build shared targets" ON)
OPTION(BUILD_STATIC "Build static targets" ON)

set (LEVELDB_MAJOR_VERSION 1)
set (LEVELDB_MINOR_VERSION 20)
set (LEVELDB_PATCH_VERSION 0)
set (LEVELDB_VERSION
  ${LEVELDB_MAJOR_VERSION}.${LEVELDB_MINOR_VERSION}.${LEVELDB_PATCH_VERSION})

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads REQUIRED)

set(LEVELDB_SOURCES db/builder.cc db/c.cc db/dbformat.cc db/db_impl.cc
  db/db_iter.cc db/dumpfile.cc db/filename.cc db/log_reader.cc
  db/log_writer.cc db/memtable.cc db/repair.cc db/table_cache.cc
  db/version_edit.cc db/version_set.cc db/write_batch.cc
  table/block_builder.cc table/block.cc table/filter_block.cc
  table/format.cc table/iterator.cc table/merger.cc
  table/table_builder.cc table/table.cc table/two_level_iterator.cc
  util/arena.cc util/bloom.cc util/cache.cc util/coding.cc
  util/comparator.cc util/crc32c.cc util/env.cc util/env_posix.cc
  util/filter_policy.cc util/hash.cc util/histogram.cc util/logging.cc
  util/options.cc util/status.cc 
  port/port_posix.cc
  port/port_posix_sse.cc) 
set(DBINCLUDES
  db/builder.h
  db/dbformat.h
  db/db_impl.h
  db/db_iter.h
  db/filename.h
  db/log_reader.h
  db/log_writer.h
  db/memtable.h
  db/table_cache.h
  db/version_edit.h
  db/version_set.h
  db/write_batch_internal.h
  )
set(LEVELDB_INCLUDES
  leveldb/cache.h
  leveldb/c.h
  leveldb/comparator.h
  leveldb/db.h
  leveldb/env.h
  leveldb/filter_policy.h
  leveldb/iterator.h
  leveldb/options.h
  leveldb/slice.h
  leveldb/status.h
  leveldb/table_builder.h
  leveldb/table.h
  leveldb/write_batch.h
  )
set(TABLE_INCLUDES
  table/block_builder.h
  table/block.h
  table/filter_block.h
  table/format.h
  table/iterator_wrapper.h
  table/merger.h
  table/two_level_iterator.h
)
set(UTIL_INCLUDES
  util/arena.h
  util/coding.h
  util/crc32c.h
  util/env_posix_test_helper.h
  util/hash.h
  util/histogram.h
  util/logging.h
  util/mutexlock.h
  util/posix_logger.h
  util/random.h)


if(${CMAKE_COMPILER_IS_GNUCXX})
  option(USE_GLIBCMEMCMP "Whether to disable gcc builtin memcmp" ON)
endif()
if(USE_GLIBCMEMCMP)
  add_compile_options(-fno-builtin-memcmp)
endif()
dbgmsg("CMAKE_SYSTEM_NAME")
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  add_definitions(-DOS_MACOSX)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  dbgmsg("CMAKE_SYSTEM_NAME")
  add_definitions(-DOS_LINUX)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "IOS")
  add_definitions(-DOS_MACOSX)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "OS_ANDROID_CROSSCOMPILE")
  add_definitions(-D_REENTRANT -DOS_ANDROID -DLEVELDB_PLATFORM_POSIX)
endif()
add_definitions(-DLEVELDB_PLATFORM_POSIX -DLEVELDB_ATOMIC_PRESENT -DSNAPPY)
add_compile_options(-march=native -mtune=native)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/include)

# if(BUILD_SHARED)
#   add_library(leveldb SHARED ${LEVELDB_SOURCES})
#   add_library(memenv helpers/memenv/memenv.cc)
# endif()
# if(BUILD_STATIC)
#   add_library(leveldbstatic STATIC ${LEVELDB_SOURCES})
#   set_target_properties(leveldbstatic PROPERTIES OUTPUT_NAME leveldb)
#   add_library(memenv STATIC helpers/memenv/memenv.cc)
# endif()
add_lib(leveldb BUILD_SHARED BUILD_STATIC ${LEVELDB_SOURCES})
add_lib(memenv  BUILD_SHARED BUILD_STATIC helpers/memenv/memenv.cc)
add_executable(db_bench db/db_bench.cc util/testutil.cc)
target_link_libraries(db_bench leveldb -lsnappy Threads::Threads)
add_executable(db_bench_static db/db_bench util/testutil.cc)
target_link_libraries(db_bench_static leveldb_static -lsnappy Threads::Threads)
set_target_properties(db_bench_static PROPERTIES OUTPUT_NAME db_bench
  RUNTIME_OUTPUT_DIRECTORY out-static
  )

add_executable(leveldbutil db/leveldbutil.cc)
target_link_libraries(leveldbutil leveldb -lsnappy Threads::Threads)
add_executable(leveldbutil_static db/leveldbutil.cc)
target_link_libraries(leveldbutil_static leveldb_static -lsnappy Threads::Threads)
set_target_properties(leveldbutil_static PROPERTIES OUTPUT_NAME leveldbutil
  RUNTIME_OUTPUT_DIRECTORY out-static)
install(TARGETS leveldb leveldb_static memenv memenv_static db_bench db_bench_static leveldbutil leveldbutil_static
  LIBRARY DESTINATION lib/
  ARCHIVE DESTINATION lib/
  RUNTIME DESTINATION bin/
  )
install(DIRECTORY include/leveldb DESTINATION include)
