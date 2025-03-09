cmake_minimum_required(VERSION 3.20)
project(swift-audiogame-libs)






include(FetchContent)
FetchContent_Declare(
  sdl
  GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
  GIT_TAG release-3.2.8
)
FetchContent_MakeAvailable(sdl)


