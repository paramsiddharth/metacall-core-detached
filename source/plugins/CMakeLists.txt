#Check if extension loader is enabled
if(NOT OPTION_BUILD_LOADERS OR NOT OPTION_BUILD_LOADERS_EXT OR NOT OPTION_BUILD_EXTENSIONS)
	return()
endif()

# Plugins options
option(OPTION_BUILD_PLUGINS_BACKTRACE "Build cross-platform backtrace plugin." ON)

# Plugin sub-projects
add_subdirectory(backtrace_plugin)

# Install plugin directory
install(DIRECTORY ${PROJECT_OUTPUT_DIR}/plugins
	DESTINATION ${INSTALL_LIB}
	PATTERN "test[-_]*" EXCLUDE
)
