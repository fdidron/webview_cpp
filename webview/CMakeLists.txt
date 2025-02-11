add_library(webview webview.cpp)

if (UNIX AND NOT APPLE)
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(gtk REQUIRED gtk+-3.0 IMPORTED_TARGET)
  pkg_check_modules(webkit REQUIRED webkit2gtk-4.0 IMPORTED_TARGET)
  target_link_libraries(webview PkgConfig::gtk PkgConfig::webkit)
endif()

if (APPLE)
  find_package(PkgConfig REQUIRED)
  target_link_libraries(webview "-framework Webkit")
endif()

if (WIN32)
  find_program(NUGET_EXE NAMES nuget)
  if (NOT NUGET_EXE)
    message("NUGET.EXE not found.")
    message(FATAL_ERROR "Please install this executable, and run CMake again.")
  endif()

  exec_program(${NUGET_EXE} ARGS install "Microsoft.Web.WebView2" -Version 1.0.1150.38
      -ExcludeVersion -OutputDirectory ${CMAKE_BINARY_DIR}/packages)
  exec_program(${NUGET_EXE} ARGS install "Microsoft.Windows.ImplementationLibrary" -Version 1.0.220201.1
      -ExcludeVersion -OutputDirectory ${CMAKE_BINARY_DIR}/packages)
  
  if(${CMAKE_SIZEOF_VOID_P} STREQUAL 8)
    set(architecture x64)
  else()
    set(architecture x86)
  endif()

  target_include_directories(webview
    PUBLIC ${CMAKE_BINARY_DIR}/packages/Microsoft.Web.WebView2/build/native/include
    PUBLIC ${CMAKE_BINARY_DIR}/packages/Microsoft.Windows.ImplementationLibrary/include
  )

  target_link_libraries(webview
    PUBLIC ${CMAKE_BINARY_DIR}/packages/Microsoft.Web.WebView2/build/native/${architecture}/WebView2LoaderStatic.lib)
endif()
