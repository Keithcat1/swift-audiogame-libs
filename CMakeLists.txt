cmake_minimum_required(VERSION 3.20)
project(swift-audiogame-libs)

function(write_link_libraries output_file)
    file(WRITE "${output_file}" "") # Clear the file first
    foreach(target IN LISTS ARGN)
        if (TARGET ${target})
            get_target_property(libs ${target} INTERFACE_LINK_LIBRARIES)
            get_target_property(priv_libs ${target} LINK_LIBRARIES)
            if (libs)
                foreach(lib IN LISTS libs)
                    file(APPEND "${output_file}" "${lib}\n")
                endforeach()
            endif()
            if (priv_libs)
                foreach(lib IN LISTS priv_libs)
                    file(APPEND "${output_file}" "${lib}\n")
                endforeach()
            endif()
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
  GIT_TAG v1.1.0
)
FetchContent_MakeAvailable(speech_core)

write_link_libraries("../link_targets.txt" SpeechCore sdl3-static)