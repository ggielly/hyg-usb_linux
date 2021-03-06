### CMake file for hyg-usb-linux
###
###
###   - Jan 2016: Initial version <bulot@macareux-labs.com>
###
###
###############################################################################


cmake_minimum_required (VERSION 2.6)

project (hyg-usb)
set (hyg-usb_VERSION_MAJOR 0)
set (hyg-usb_VERSION_MINOR 1)

set( CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} /lib /usr/lib /usr/local/lib )
set( CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} /usr/include /usr/local/include )

set( CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake" ) 

set ( INSTALL_BIN_PATH bin )
set ( INSTALL_LIB_PATH lib )
set ( INSTALL_AR_PATH  lib )
set ( INSTALL_HEADERS_PATH include )


# Make relative paths absolute (needed later on)

foreach(p LIB BIN HEADERS AR)
        set(var INSTALL_${p}_PATH)
        if(NOT IS_ABSOLUTE "${${var}}")
                set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
        endif()
endforeach()


option ( HYG_INSTALL_MAN "Generate and install manpages ?" ON )
option ( HYG_STATIC_LINK "Generate statically linked exec ?" OFF )


### Search for Lib/Headers

include( GNUInstallDirs )

if ( HYG_STATIC_LINK )

	set(CMAKE_EXE_LINKER_FLAGS -static)
	set(CMAKE_FIND_LIBRARY_SUFFIXES .a)

	find_package( libpthread-static REQUIRED )
	find_package( librt-static REQUIRED )

endif (HYG_STATIC_LINK )

find_package( libusb-1.0 REQUIRED)

### Config File

configure_file ( "${PROJECT_SOURCE_DIR}/config.h.in"
		 "${PROJECT_SOURCE_DIR}/src/config.h" )

### Set include directories 
        
include_directories( "${CMAKE_CURRENT_SOURCE_DIR}/lib/include"
	             "${CMAKE_CURRENT_BINARY_DIR}"
		      ${LIBUSB_1_INCLUDE_DIRS} 
)

### Set link directories

# link_directories( "${VISION_LIBRARY_DIR}" )


### File lists

file( GLOB src "src/*.c"  )
file( GLOB hdr "src/*.h"  "include/*.h" )


### Targets

add_executable( "hyg-usb" ${src} ${hdr} ) 

### Linking

if (HYG_STATIC_LINK )

	set_target_properties(hyg-usb PROPERTIES LINK_SEARCH_START_STATIC 1)
	set_target_properties(hyg-usb PROPERTIES LINK_SEARCH_END_STATIC 1)
	
	target_link_libraries( "hyg-usb" 
				${LIBUSB_1_LIBRARIES}
				${LIBPTHREAD_LIBRARIES}
				${LIBRT_LIBRARIES}
	)

else (HYG_STATIC_LINK )

	target_link_libraries( "hyg-usb" ${LIBUSB_1_LIBRARIES} )

endif (HYG_STATIC_LINK )

### Install 

install( TARGETS "hyg-usb"
	    RUNTIME DESTINATION ${INSTALL_BIN_PATH}  COMPONENT main
	    LIBRARY DESTINATION ${INSTALL_LIB_PATH}  COMPONENT main
            ARCHIVE DESTINATION ${INSTALL_AR_PATH}   COMPONENT main )


### Manpage generation and install
      
if ( HYG_INSTALL_MAN )
	add_subdirectory( "man" )
endif ( HYG_INSTALL_MAN )
