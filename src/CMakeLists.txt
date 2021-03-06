include(AMXConfig)
include(AddSAMPPlugin)

if(NOT WIN32 AND MYSQLCAPI_LIBRARY_STATIC)
	option(BUILD_STATIC "Build static library" OFF)
endif()

set(SAMP_SDK_ROOT "${PROJECT_SOURCE_DIR}/libs/sdk")
find_package(SAMPSDK REQUIRED)

set(MYSQL_PLUGIN_CONFIG_INCLUDE_DIR "${CMAKE_CURRENT_BINARY_DIR}/config_headers")
include_directories(
	${SAMPSDK_INCLUDE_DIR}
	${MYSQL_PLUGIN_CONFIG_INCLUDE_DIR}
	${PROJECT_SOURCE_DIR}/libs/samp-log-core/include
)

configure_file(
	"../a_mysql.inc.in"
	"a_mysql.inc"
	@ONLY
)

configure_file(
	"version.hpp.in"
	"${MYSQL_PLUGIN_CONFIG_INCLUDE_DIR}/version.hpp"
	@ONLY
)

add_samp_plugin(mysql
	${SAMPSDK_DIR}/amxplugin.cpp
	${SAMPSDK_DIR}/amxplugin2.cpp
	${SAMPSDK_DIR}/amx/getch.c
	CCallback.cpp
	CCallback.hpp
	CConnection.cpp
	CConnection.hpp
	CDispatcher.cpp
	CDispatcher.hpp
	CError.hpp
	CHandle.cpp
	CHandle.hpp
	CLog.cpp
	CLog.hpp
	COptions.cpp
	COptions.hpp
	COrm.cpp
	COrm.hpp
	CQuery.cpp
	CQuery.hpp
	CResult.cpp
	CResult.hpp
	natives.cpp
	natives.hpp
	CSingleton.hpp
	main.cpp
	misc.hpp
	mysql.hpp
	sdk.hpp
	types.hpp
	sscanf.cpp
	sscanf.hpp
	plugin.def
)

target_include_directories(mysql PRIVATE
	${Boost_INCLUDE_DIR}
	${MYSQLCAPI_INCLUDE_DIR}
	${LOGCORE_INCLUDE_DIR}
)

target_compile_features(mysql PUBLIC
	cxx_auto_type
	cxx_decltype
	cxx_defaulted_functions
	cxx_default_function_template_args
	cxx_delegating_constructors
	cxx_deleted_functions
	cxx_enum_forward_declarations
	cxx_explicit_conversions
	cxx_extended_friend_declarations
	cxx_lambdas
	cxx_inheriting_constructors
	cxx_noexcept
	cxx_nonstatic_member_init
	cxx_nullptr
	cxx_range_for
	cxx_reference_qualified_functions
	cxx_right_angle_brackets
	cxx_rvalue_references
	cxx_strong_enums
	cxx_variadic_templates
)

add_dependencies(mysql log-core fmt)

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -DNOMINMAX -D_ENABLE_ATOMIC_ALIGNMENT_FIX)
endif()

target_link_libraries(mysql log-core fmt)

if(UNIX)
	if(NOT APPLE)
		target_link_libraries(mysql rt)
	endif()

	set_target_properties(mysql PROPERTIES
		INSTALL_RPATH "$ORIGIN/..:$ORIGIN/")
endif()

if(BUILD_STATIC)
	target_link_libraries(mysql "${MYSQLCAPI_LIBRARY_STATIC}")
else()
	target_link_libraries(mysql "${MYSQLCAPI_LIBRARY}")
endif()

install(TARGETS mysql DESTINATION "plugins/")
get_target_property(LOGCORE_LOCATION log-core LOCATION)
install(FILES "${LOGCORE_LOCATION}" DESTINATION "./")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/a_mysql.inc" DESTINATION "pawno/include/")
install(FILES "${PROJECT_SOURCE_DIR}/LICENSE" DESTINATION "./")
if(WIN32 AND MYSQLCAPI_LIBRARY_SHARED)
	install(FILES "${MYSQLCAPI_LIBRARY_SHARED}" DESTINATION "./")
endif()

set(CPACK_PACKAGE_VERSION ${MYSQL_PLUGIN_VERSION})
if(WIN32)
	set(CPACK_GENERATOR ZIP)
else()
	set(CPACK_GENERATOR TGZ)
endif()

include(CPack)
