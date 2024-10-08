# cmake version verification
cmake_minimum_required(VERSION 3.30.1)

# support for print
include(CMakePrintHelpers)

# author’s details
set(CPACK_PACKAGE_VENDOR "Rajiv Sithiravel")
set(CPACK_NSIS_CONTACT "rajiv.sithiravel@gmail.com")

# project name
project(mx20datareader LANGUAGES CXX C VERSION 0.0.1)
cmake_print_variables(PROJECT_NAME)

# for consistency lowercases for project name
string(TOLOWER ${PROJECT_NAME} PROJECT_NAME_LOWER)

# enable all warnings during compile process
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")

# enable the CXX standard version
set(CMAKE_CXX_STANDARD 20)
cmake_print_variables(CMAKE_CXX_STANDARD)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# set build type
if(NOT CMAKE_BUILD_TYPE)
	set(DEFAULT_BUILD_TYPE "Release")
    set(CMAKE_BUILD_TYPE ${DEFAULT_BUILD_TYPE})
endif()

# set architecture to default if not set
if(NOT ARCH)
	set(DEFAULT_ARCH "x86-64")
    set(ARCH ${DEFAULT_ARCH})
endif()

# detect the operating system 
set(operating_systems 
"BlueGeneQ-static"
"BSDOS"
"Catamount"
"CrayLinuxEnvironment"
"CYGWIN"
"Darwin"
"DOS"
"DragonFly"
"eCos"
"Emscripten"
"Euros"
"FreeBSD"
"Fuchsia"
"Generic-ADSP"
"Generic-ELF"
"Generic"
"GHS-MULTI"
"GNU"
"Haiku"
"HP-UX"
"iOS"
"kFreeBSD"
"Linux"
"Midipix"
"MirBSD"
"MP-RAS"
"MSYS"
"NetBSD"
"OpenBSD"
"OpenVMS"
"OS2"
"OSF1"
"QNX"
"RISCos"
"SCO_SV"
"SerenityOS"
"SINIX"
"SunOS"
"syllable"
"Tru64"
"tvOS"
"ULTRIX"
"UNIX_SV"
"UnixWare"
"visionOS"
"watchOS"
"Windows"
"WindowsCE"
"WindowsPhone"
"WindowsStore"
"Xenix"
)

set(os_detected_status "false")
foreach(osysm ${operating_systems})
	if(${CMAKE_SYSTEM_NAME} MATCHES ${osysm})		
		cmake_print_variables(CMAKE_SYSTEM)	
		cmake_print_variables(CMAKE_SYSTEM_NAME)	
		cmake_print_variables(CMAKE_SYSTEM_VERSION)	
		cmake_print_variables(CMAKE_SYSTEM_PROCESSOR)	
		cmake_print_variables(CMAKE_HOST_SYSTEM_NAME)	
		set(os_detected_status "true")
	endif()
endforeach()

if(os_detected_status MATCHES false)
	message("operating system: Unknown")
endif()

# detect compiler
set(compilers 
"Absoft"
"ADSP"
"AppleClang"
"ARMCC"
"ARMClang"
"Bruce"
"CCur"
"Clang"
"Cray"
"CrayClang"
"Borland"
"Flang"
"LLVMFlang"
"Fujitsu"
"FujitsuClang"
"G95"
"GNU"
"GHS"
"HP"
"IAR"
"Intel"
"IntelLLVM"
"LCC"
"MSVC"
"NVHPC"
"NVIDIA"
"OrangeC"
"OpenWatcom"
"PGI"
"PathScale"
"SDCC"
"SunPro"
"Tasking"
"TI"
"TIClang"
"TinyCC"
"XL"
"XLClang"
"IBMClang"
)
if(CMAKE_CXX_COMPILER_LOADED)
	set(compiler_detected_status "false")
	foreach(comps ${compilers})
		if(${CMAKE_CXX_COMPILER_ID} STREQUAL ${comps})		
			cmake_print_variables(CMAKE_CXX_COMPILER)	
			cmake_print_variables(CMAKE_CXX_COMPILER_ID)	
			cmake_print_variables(CMAKE_CXX_COMPILER_VERSION)	
			set(compiler_detected_status "true")
		endif()
	endforeach()
	
	if(compiler_detected_status MATCHES false)
		message("compiler: Unknown")
	endif()
endif()

# add compile options
if(CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
    set(CLANG 1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(CLANG 1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(GCC 1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    set(INTEL 1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	add_compile_options(/bigobj)
    set(MSVC 1)
endif()

# set bitness
if(${ARCH} STREQUAL "x86-64")
    set(BITNESS "64")
    set(BITNESS_FLAG "-m${BITNESS}")
elseif(${ARCH} STREQUAL "x86")
    set(BITNESS "32")
    set(BITNESS_FLAG "-m${BITNESS}")
elseif(${ARCH} STREQUAL "armv8")
    set(BITNESS "64")
    set(BITNESS_FLAG "")
else()
    set(BITNESS "32")
    set(ARCH "x86")
    set(BITNESS_FLAG "-m${BITNESS}")
    message("Unknown architecture selected, defaulting to x86")
endif()

if(MSVC)
    # User cannot specify bitness with MSVC, so set it to whatever the generator is.
    string(TOLOWER ${CMAKE_GENERATOR} GENERATOR)
    if(GENERATOR MATCHES ".*win64.*")
        set(BITNESS "64")
    else()
        set(BITNESS "32")
    endif()
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Ox")

elseif(GCC OR CLANG)
    # Treat warning return-type as error to avoid undefined behaviour
    # when a non-void function does not return a value.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BITNESS_FLAG} -std=c++11 -Werror=return-type")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall -Wno-long-long")

elseif(INTEL)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BITNESS_FLAG} -std=c++11")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
endif()

string(COMPARE EQUAL "Windows" ${CMAKE_SYSTEM_NAME} WINDOWS)
if(WINDOWS) 
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /STACK:100000000")
option(BUILD_SHARED_LIBS "Build using shared libraries" OFF)
add_compile_definitions(_SILENCE_EXPERIMENTAL_FILESYSTEM_DEPRECATION_WARNING _ENFORCE_MATCHING_ALLOCATORS=0 WINDOWS_ENABLE_CPLUSPLUS)
endif(WINDOWS) 

# extract third-party libraries from Conan 
find_package(boost REQUIRED)
find_package(Eigen3 REQUIRED)

# add sources 
set(SRC_LIST
	${PROJECT_SOURCE_DIR}/src/ac3.c    
	${PROJECT_SOURCE_DIR}/src/accessunit.c    
	${PROJECT_SOURCE_DIR}/src/adts.c    
	${PROJECT_SOURCE_DIR}/src/audio.c    
	${PROJECT_SOURCE_DIR}/src/avs.c    
	${PROJECT_SOURCE_DIR}/src/bitdata.c    
	${PROJECT_SOURCE_DIR}/src/es.c    
	${PROJECT_SOURCE_DIR}/src/ethernet.c    
	${PROJECT_SOURCE_DIR}/src/filter.c    
	${PROJECT_SOURCE_DIR}/src/fmtx.c    
	${PROJECT_SOURCE_DIR}/src/h222.c    
	${PROJECT_SOURCE_DIR}/src/h262.c    
	${PROJECT_SOURCE_DIR}/src/ipv4.c    
	${PROJECT_SOURCE_DIR}/src/l2audio.c    
	${PROJECT_SOURCE_DIR}/src/misc.c    
	${PROJECT_SOURCE_DIR}/src/nalunit.c    
	${PROJECT_SOURCE_DIR}/src/pcap.c    
	${PROJECT_SOURCE_DIR}/src/pes.c    
	${PROJECT_SOURCE_DIR}/src/pidint.c    
	${PROJECT_SOURCE_DIR}/src/printing.c    
	${PROJECT_SOURCE_DIR}/src/ps.c    
	${PROJECT_SOURCE_DIR}/src/reverse.c    
	${PROJECT_SOURCE_DIR}/src/ts.c    
	${PROJECT_SOURCE_DIR}/src/tsplay_innards.c    
	${PROJECT_SOURCE_DIR}/src/tswrite.c  
)

set(EXAMPLE_SRC_LIST
	${PROJECT_SOURCE_DIR}/test/tsreport.c 
)

# adds executable target with given name
add_executable(${PROJECT_NAME_LOWER} 
	${SRC_LIST}
	${EXAMPLE_SRC_LIST}	
)

if(WINDOWS) 				
target_compile_options(${PROJECT_NAME_LOWER} PRIVATE "/Od")   
endif(WINDOWS)

target_include_directories(${PROJECT_NAME_LOWER} PRIVATE 
	${PROJECT_SOURCE_DIR}/include
) 

target_link_libraries(${PROJECT_NAME_LOWER} 
	boost::boost 
	Eigen3::Eigen 
)


# install
install(TARGETS ${PROJECT_NAME_LOWER})

# print the configuration info
message("Configuring ${PROJECT_NAME_LOWER} version ${VERSION} in ${CMAKE_BUILD_TYPE} mode for ${ARCH} (${BITNESS} bit)")
message("Compiler flags: ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE}}")

