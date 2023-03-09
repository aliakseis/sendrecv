cmake_minimum_required(VERSION 3.10)

# プロジェクト名と使用する言語を設定
project(sendrecv C CXX)

set(CMAKE_CXX_STANDARD 17)


# pkg-config を使ってモジュールを読み込む
find_package(PkgConfig)

# gstreamer-1.0 の存在チェック
pkg_check_modules(GSTREAMER REQUIRED 
  glib-2.0
  json-glib-1.0
  libsoup-2.4
  gstreamer-1.0 
  gstreamer-sdp-1.0
  gstreamer-rtp-1.0
  gstreamer-webrtc-1.0)

# gstreamer のヘッダーファイルへのパスを表示
message("GSTREAMER_INCLUDE_DIRS: ${GSTREAMER_INCLUDE_DIRS}")

# gstreamer のライブラリへのパスを表示
message("GSTREAMER_LIBRARY_DIRS: ${GSTREAMER_LIBRARY_DIRS}")


# gstreamermm のライブラリのパスを設定
link_directories(${GSTREAMER_LIBRARY_DIRS})

find_package(crossguid CONFIG REQUIRED)

find_package(curl CONFIG REQUIRED)

# コンパイルフラグを設定
#set(CMAKE_C_FLAGS "-Wall")
#set(CMAKE_C_FLAGS_DEBUG "-g3 -O0 -pg")
#set(CMAKE_C_FLAGS_RELEASE "-O2 -s -DNDEBUG -march=native")
#set(CMAKE_C_FLAGS_RELWITHDEBINFO "-g3 -Og -pg")
#set(CMAKE_C_FLAGS_MINSIZEREL "-Os -s DNDEBUG -march=native")

#set(CMAKE_CXX_FLAGS "-Wall -std=c++17")
#set(CMAKE_CXX_FLAGS_DEBUG "-g -DDEBUG_BUILD")
#set(CMAKE_CXX_FLAGS_RELEASE "-O2 -s -DNDEBUG -march=native")
#set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-g3 -Og -pg")
#set(CMAKE_CXX_FLAGS_MINSIZEREL "-Os -s DNDEBUG -march=native")

# 実行ファイルの作成
add_executable(sendrecv 
  src/main.cpp src/http.cpp src/http.h)

# gstreamer ヘッダーへのパスを設定
target_include_directories(sendrecv  PUBLIC ${GSTREAMER_INCLUDE_DIRS})

# gstreamermm ライブラリの設定
target_link_libraries(sendrecv  ${GSTREAMER_LIBRARIES} crossguid CURL::libcurl)
#target_link_libraries(sendrecv  pthread)

# gstreamer のコンパイルオプションを設定
target_compile_options(sendrecv  PUBLIC ${GSTREAMER_CFLAGS_OTHER})

if(MSVC)
	set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/bin")

	set(BINARY_INSTALL_DIR .)

	install(TARGETS ${PROJECT_NAME} DESTINATION ${BINARY_INSTALL_DIR})


	set(BINARY_FILE "${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.exe")

	get_filename_component(CMAKE_TOOLCHAIN_DIRECTORY ${CMAKE_TOOLCHAIN_FILE} DIRECTORY)

	#install(CODE "execute_process(COMMAND \"powershell\" \"-noprofile\" \"-executionpolicy\" \"Bypass\" \"-file\" \"${CMAKE_TOOLCHAIN_DIRECTORY}/msbuild/applocal.ps1\" \"-targetBinary\" \"${BINARY_FILE}\" \"-installedDir\" \"$ENV{GSTREAMER_ROOT_X86_64}/bin\" \"-OutVariable\" \"out\")")


	if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	    set( CMAKE_INSTALL_PLATFORM "x64-windows" )
	else ()
	    set( CMAKE_INSTALL_PLATFORM "x86-windows" )
	endif ()

	install(CODE "execute_process(COMMAND \"powershell\" \"-noprofile\" \"-executionpolicy\" \"Bypass\" \"-file\" \"${CMAKE_TOOLCHAIN_DIRECTORY}/msbuild/applocal.ps1\" \"-targetBinary\" \"${BINARY_FILE}\" \"-installedDir\" \"${CMAKE_TOOLCHAIN_DIRECTORY}/../../installed/${CMAKE_INSTALL_PLATFORM}/bin\" \"-OutVariable\" \"out\")")

	# Install CRT
	set(CMAKE_INSTALL_SYSTEM_RUNTIME_DESTINATION .)
	#if(DEVELOPER_FEATURES)
	#	set(CMAKE_INSTALL_DEBUG_LIBRARIES ON)
	#endif(DEVELOPER_FEATURES)
	set(CMAKE_INSTALL_UCRT_LIBRARIES TRUE)
	set(CMAKE_INSTALL_OPENMP_LIBRARIES TRUE)
	include (InstallRequiredSystemLibraries)

	#if (QT_QMAKE_EXECUTABLE)
	#	get_filename_component(QT_BIN_DIRECTORY ${QT_QMAKE_EXECUTABLE} DIRECTORY)
	#	install(CODE "execute_process(COMMAND \"${QT_BIN_DIRECTORY}/windeployqt\"  \"${BINARY_FILE}\" WORKING_DIRECTORY \"${QT_BIN_DIRECTORY}\")")
	#else()
	#	install(CODE "execute_process(COMMAND \"windeployqt\"  \"${BINARY_FILE}\" )")
	#endif()


	#if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	#    install(DIRECTORY "$ENV{GSTREAMER_ROOT_X86_64}/bin/" DESTINATION .)
	#else ()
	#    install(DIRECTORY "$ENV{GSTREAMER_ROOT_X86}/bin/" DESTINATION .)
	#endif ()

	if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	    install(DIRECTORY "$ENV{GSTREAMER_ROOT_X86_64}/" DESTINATION gst PATTERN "*.pdb" EXCLUDE)
	else ()
	    install(DIRECTORY "$ENV{GSTREAMER_ROOT_X86}/" DESTINATION gst PATTERN "*.pdb" EXCLUDE)
	endif ()

	install(FILES run.cmd DESTINATION .)

endif()
