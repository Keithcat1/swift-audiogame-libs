cmake_minimum_required(VERSION 3.20)
project(swift-audiogame-libs)


# Function to recursively get all linked libraries
function(get_all_system_libraries OUTPUT_FILE)
  set(ALL_LIBRARIES "")
  set(PROCESSED_TARGETS "")
  
  # Process each target in the list
  foreach(TARGET_NAME ${ARGN})
    # Skip if not a valid target
    if(NOT TARGET ${TARGET_NAME})
      message(STATUS "Skipping non-target: ${TARGET_NAME}")
      continue()
    endif()
    
    # Recursively collect all link libraries for this target
    _get_all_link_libraries_recursive(${TARGET_NAME} ALL_LIBRARIES PROCESSED_TARGETS)
  endforeach()
  
  # Remove duplicates
  list(REMOVE_DUPLICATES ALL_LIBRARIES)
  
  # Process libraries to get clean system library names
  set(CLEAN_SYSTEM_LIBRARIES "")
  foreach(lib ${ALL_LIBRARIES})
    # Skip CMake targets (which are not system libraries)
    if(TARGET ${lib})
      continue()
    endif()
    
    # Handle generator expressions
    string(GENEX_STRIP "${lib}" lib_no_genex)
    
    # Skip empty strings after genex removal
    if("${lib_no_genex}" STREQUAL "")
      continue()
    endif()
    
    # Extract library name for different platforms
    # Windows: typically full paths or lib names
    # Unix: could be -l... form or full paths
    
    # Check if it's a path
    if(EXISTS "${lib_no_genex}")
      get_filename_component(lib_name "${lib_no_genex}" NAME_WE)
      # Remove lib prefix on Unix-like systems
      string(REGEX REPLACE "^lib" "" lib_name "${lib_name}")
      list(APPEND CLEAN_SYSTEM_LIBRARIES "${lib_name}")
    else()
      # Handle -l format on Unix
      string(REGEX REPLACE "^-l" "" lib_name "${lib_no_genex}")
      # Handle standard library names on all platforms
      if(NOT "${lib_name}" STREQUAL "${lib_no_genex}")
        list(APPEND CLEAN_SYSTEM_LIBRARIES "${lib_name}")
      else()
        # Handle Windows-style library names (without path)
        string(REGEX REPLACE "\\.lib$" "" win_lib_name "${lib_no_genex}")
        if(NOT "${win_lib_name}" STREQUAL "${lib_no_genex}")
          list(APPEND CLEAN_SYSTEM_LIBRARIES "${win_lib_name}")
        else()
          # Just use the stripped name if no other rules match
          list(APPEND CLEAN_SYSTEM_LIBRARIES "${lib_no_genex}")
        endif()
      endif()
    endif()
  endforeach()
  
  # Sort the list for consistency
  list(SORT CLEAN_SYSTEM_LIBRARIES)
  
  # Write to file
  file(WRITE "${OUTPUT_FILE}" "")
  foreach(lib ${CLEAN_SYSTEM_LIBRARIES})
    file(APPEND "${OUTPUT_FILE}" "${lib}\n")
  endforeach()
  
  message(STATUS "System libraries written to ${OUTPUT_FILE}")
endfunction()


# Private helper function for recursion - now includes private libraries
function(_get_all_link_libraries_recursive target result processed_list)
  # Resolve aliases
  get_target_property(aliased_target ${target} ALIASED_TARGET)
  if(aliased_target)
    set(actual_target ${aliased_target})
  else()
    set(actual_target ${target})
  endif()
  
  # Check if we've already processed this target to avoid infinite recursion
  list(FIND ${processed_list} ${actual_target} target_index)
  if(NOT ${target_index} EQUAL -1)
    return()
  endif()
  
  # Mark as processed by adding to the list
  list(APPEND ${processed_list} ${actual_target})
  set(${processed_list} ${${processed_list}} PARENT_SCOPE)
  
  # Get all types of link libraries (private, interface, and public)
  
  # LINK_LIBRARIES contains all libraries (public and private)
  get_target_property(LINK_LIBRARIES ${actual_target} LINK_LIBRARIES)
  
  # INTERFACE_LINK_LIBRARIES contains public/interface libraries
  get_target_property(INTERFACE_LIBRARIES ${actual_target} INTERFACE_LINK_LIBRARIES)
  
  # IMPORTED_LINK_DEPENDENT_LIBRARIES often contains private dependencies for imported targets
  get_target_property(IMPORTED_LIBRARIES ${actual_target} IMPORTED_LINK_DEPENDENT_LIBRARIES)
  
  # LINK_INTERFACE_LIBRARIES is older way to specify interface libraries
  get_target_property(LINK_INTERFACE_LIBRARIES ${actual_target} LINK_INTERFACE_LIBRARIES)
  
  # Process all non-null library lists
  foreach(lib_list LINK_LIBRARIES INTERFACE_LIBRARIES IMPORTED_LIBRARIES LINK_INTERFACE_LIBRARIES)
    if(${lib_list})
      foreach(lib ${${lib_list}})
        if(TARGET ${lib})
          # For target libraries, recurse to get their dependencies
          _get_all_link_libraries_recursive(${lib} ${result} ${processed_list})
        else()
          # For non-target libraries, add to result
          list(APPEND ${result} ${lib})
        endif()
      endforeach()
    endif()
  endforeach()
  
  # Return the result
  set(${result} ${${result}} PARENT_SCOPE)
endfunction()


include(FetchContent)

option(BUILD_SHARED_LIBS "Build using shared libraries" OFF)

FetchContent_Declare(
  sdl
  GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
  GIT_TAG release-3.2.8
    OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(sdl)


FetchContent_Declare(
  speech_core
  GIT_REPOSITORY https://github.com/keithcat1/SpeechCore.git
  GIT_TAG v1.1.1
)
FetchContent_MakeAvailable(speech_core)


# required dependency of vorbis
    
FetchContent_Declare(
    ogg
    GIT_REPOSITORY https://github.com/xiph/ogg.git
    GIT_TAG v1.3.5
    OVERRIDE_FIND_PACKAGE # force vorbis to find it
)
FetchContent_MakeAvailable(ogg)


FetchContent_Declare(
    vorbis
    GIT_REPOSITORY https://github.com/xiph/vorbis.git
    GIT_TAG v1.3.7
)
FetchContent_MakeAvailable(vorbis)


FetchContent_Declare(
    opus
    GIT_REPOSITORY https://github.com/xiph/opus.git
    GIT_TAG v1.5.2
    OVERRIDE_FIND_PACKAGE   
)
FetchContent_MakeAvailable(opus)


set(OP_DISABLE_HTTP TRUE)
                    set(OP_DISABLE_DOCS TRUE)
                    set(OP_DISABLE_EXAMPLES TRUE)

FetchContent_Declare(
    opusfile
    GIT_REPOSITORY https://github.com/xiph/opusfile.git
    # only the master branch of opusfile has CMakeSupport

)
FetchContent_MakeAvailable(opusfile)




FetchContent_Declare(
  miniaudio
  GIT_REPOSITORY https://github.com/mackron/miniaudio.git
  GIT_TAG 0.11.22
)
FetchContent_MakeAvailable(miniaudio)


get_all_system_libraries("link_targets.txt" SpeechCore SDL3::SDL3 vorbis opus miniaudio)