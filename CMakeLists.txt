cmake_minimum_required(VERSION 3.20)
project(swift-audiogame-libs)


function(write_link_libraries output_file)
  # Clear the file first
  file(WRITE "${output_file}" "")

  foreach(target IN LISTS ARGN)
    if (TARGET ${target})
      # Get both public and private link libraries.
      get_target_property(priv_libs ${target} LINK_LIBRARIES)
      get_target_property(pub_libs ${target} INTERFACE_LINK_LIBRARIES)
      
      # Combine both lists (if defined)
      set(all_libs "")
      if (priv_libs)
        list(APPEND all_libs ${priv_libs})
      endif()
      if (pub_libs)
        list(APPEND all_libs ${pub_libs})
      endif()

      foreach(lib IN LISTS all_libs)
        # Remove $<LINK_ONLY:...> if present.
        string(REGEX REPLACE "\\$<LINK_ONLY:([^>]+)>" "\\1" plain_lib "${lib}")
        file(APPEND "${output_file}" "${plain_lib}\n")
      endforeach()
    else()
      message(WARNING "Target '${target}' does not exist!")
    endif()
  endforeach()
endfunction()


include(FetchContent)

FetchContent_Declare(
  sdl
  GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
  GIT_TAG release-3.2.8
)
FetchContent_MakeAvailable(sdl)


FetchContent_Declare(
  speech_core
  GIT_REPOSITORY https://github.com/keithcat1/SpeechCore.git
  GIT_TAG v1.1.1
)
FetchContent_MakeAvailable(speech_core)

write_link_libraries("link_targets.txt" SpeechCore sdl)