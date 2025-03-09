project(swift-audiogame-libs)
cmake_minimum_required(VERSION 3.20)





include(FetchContent)
FetchContent_Declare(
  sdl
  GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
  GIT_TAG release-3.2.8
)
FetchContent_MakeAvailable(sdl)


