set(SCRIPTS
    "Scripts/experience.js"
    "Scripts/playground_runner.js")
set(BABYLONSCRIPTS
    "../BabylonScripts/babylon.glTF2FileLoader.js"
    "../BabylonScripts/babylon.max.js"
    "../BabylonScripts/babylonjs.materials.js")
    
set(SOURCES
    ${SCRIPTS}
    ${BABYLONSCRIPTS}
    "Shared/InputManager.cpp"
    "Shared/InputManager.h")
if(WIN32 AND NOT WINDOWS_STORE)
    set(SOURCES
        ${SOURCES}
        "Win32/App.cpp"
        "Win32/App.h"
        "Win32/App.ico"
        "Win32/App.rc"
        "Win32/Resource.h"
        "Win32/small.ico"
        "Win32/targetver.h")
elseif(WINDOWS_STORE)
    set(APPX_FILES "UWP/Package.appxmanifest" "UWP/TemporaryKey.pfx")
    set_property(SOURCE ${APPX_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)

    set(APPX_ASSETS
        "UWP/Assets/LockScreenLogo.scale-200.png"
        "UWP/Assets/SplashScreen.scale-200.png"
        "UWP/Assets/Square44x44Logo.scale-200.png"
        "UWP/Assets/Square44x44Logo.targetsize-24_altform-unplated.png"
        "UWP/Assets/Square150x150Logo.scale-200.png"
        "UWP/Assets/StoreLogo.png"
        "UWP/Assets/Wide310x150Logo.scale-200.png")
    set_property(SOURCE ${APPX_ASSETS} PROPERTY VS_DEPLOYMENT_CONTENT 1)
    set_property(SOURCE ${APPX_ASSETS} PROPERTY VS_DEPLOYMENT_LOCATION "Assets")

    set(SOURCES
        ${SOURCES}
        ${APPX_FILES}
        ${APPX_ASSETS}
        "UWP/App.cpp"
        "UWP/App.h")

    set(RESOURCE_FILES
        ${BABYLON_NATIVE_DEPENDENCIES})

elseif(APPLE)
    find_library(JSCORE_LIBRARY JavaScriptCore)
    if(IOS)
        set(PLIST_FILE "${CMAKE_CURRENT_LIST_DIR}/iOS/Info.plist")

        set(STORYBOARD "${CMAKE_CURRENT_LIST_DIR}/iOS/Base.lproj/Main.storyboard"
                       "${CMAKE_CURRENT_LIST_DIR}/iOS/Base.lproj/LaunchScreen.storyboard")
        set(RESOURCE_FILES ${STORYBOARD} ${SCRIPTS})
        set(ADDITIONAL_LIBRARIES z)
        set(SOURCES
            ${SOURCES}
            "iOS/AppDelegate.swift"
            "iOS/ViewController.swift"
            "iOS/LibNativeBridge.h"
            "iOS/LibNativeBridge.mm")
    else()
        set(PLIST_FILE "${CMAKE_CURRENT_LIST_DIR}/macOS/Info.plist")
        set(STORYBOARD "${CMAKE_CURRENT_LIST_DIR}/macOS/Base.lproj/Main.storyboard")

        set(STORYBOARD "${CMAKE_CURRENT_LIST_DIR}/macOS/Base.lproj/Main.storyboard")
        set(RESOURCE_FILES ${STORYBOARD})

        set(SOURCES
            ${SOURCES}
            "macOS/main.m"
            "macOS/AppDelegate.m"
            "macOS/AppDelegate.h"
            "macOS/ViewController.mm"
            "macOS/ViewController.h")
    endif()

    set(ADDITIONAL_LIBRARIES ${ADDITIONAL_LIBRARIES} ${JSCORE_LIBRARY})
    set(JSSCRIPT "${CMAKE_CURRENT_LIST_DIR}/../BabylonScripts/babylon.glTF2FileLoader.js"
        "${CMAKE_CURRENT_LIST_DIR}/../BabylonScripts/babylon.max.js"
        "${CMAKE_CURRENT_LIST_DIR}/Scripts/experience.js"
        "${CMAKE_CURRENT_LIST_DIR}/Scripts/playground_runner.js")
    set(RESOURCE_FILES ${STORYBOARD} ${JSSCRIPT})
endif()

if(WIN32 OR WINDOWS_STORE)
    set(WIN32_EXECUTABLE TRUE)
    add_executable(Playground WIN32 ${SOURCES} ${RESOURCE_FILES})
else()
    add_executable(Playground ${SOURCES} ${RESOURCE_FILES})
endif()


set_property(DIRECTORY ${CMAKE_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT Playground)

source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/.." FILES ${SOURCES})

target_compile_definitions(Playground PRIVATE UNICODE)
target_compile_definitions(Playground PRIVATE _UNICODE)

if(WINDOWS_STORE)
    target_compile_options(Playground PRIVATE /ZW)
    target_compile_options(Playground PRIVATE /await)
endif()

if(WIN32)
    set(TEST_DEPENDENCY "${CMAKE_CURRENT_SOURCE_DIR}/../../Dependencies/bgfx.cmake/bx/include/compat/msvc"
        "${CMAKE_CURRENT_SOURCE_DIR}/../../Dependencies/bgfx.cmake/bgfx/include"
        "${CMAKE_CURRENT_SOURCE_DIR}/../../Dependencies/bgfx.cmake/bx/include"
        "${CMAKE_CURRENT_SOURCE_DIR}/../../Dependencies/bgfx.cmake/bimg/include")
endif()

target_include_directories(Playground PRIVATE "Source" "." ${TEST_DEPENDENCY})

target_link_libraries(Playground PRIVATE AppRuntime ${ADDITIONAL_LIBRARIES})
target_link_libraries(Playground
    PRIVATE Console
    PRIVATE NativeEngine
    PRIVATE NativeWindow
    PRIVATE ScriptLoader
    PRIVATE XMLHttpRequest)

if (WIN32)
    target_link_libraries(Playground 
        PRIVATE "shlwapi.lib")
endif()

if(APPLE)
    if(IOS)
        set_target_properties(Playground PROPERTIES
            MACOSX_BUNDLE true
            MACOSX_BUNDLE_INFO_PLIST "${PLIST_FILE}"
            XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES
            RESOURCE "${RESOURCE_FILES}"

            XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET 9.0
            XCODE_ATTRIBUTE_PRODUCT_BUNDLE_IDENTIFIER "com.BabylonNative.Playground.iOS"

            XCODE_ATTRIBUTE_ENABLE_BITCODE NO

            XCODE_ATTRIBUTE_SWIFT_VERSION "4.0"
            XCODE_ATTRIBUTE_SWIFT_OBJC_BRIDGING_HEADER "${CMAKE_CURRENT_LIST_DIR}/iOS/LibNativeBridge.h"
            XCODE_ATTRIBUTE_LD_RUNPATH_SEARCH_PATHS "@executable_path/Frameworks"
            XCODE_ATTRIBUTE_ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES YES

            # CMake seems to add a custom flag "-Wno-unknown-pragmas" to the Swift compiler. That flag is used for Clang,
            # So we need to make sure we override it with nothing here in order to compile Swift.
            XCODE_ATTRIBUTE_OTHER_SWIFT_FLAGS "")

            # Swift support
            set(CMAKE_Swift_COMPILER_FORCED TRUE)
            set(CMAKE_Swift_LANGUAGE_VERSION 4.0)
            enable_language(Swift)
    else()
        set_target_properties(Playground PROPERTIES
            MACOSX_BUNDLE true
            MACOSX_BUNDLE_INFO_PLIST "${PLIST_FILE}"
            XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES
            RESOURCE "${RESOURCE_FILES}"
            FOLDER "Playground")
    endif()
endif()

if(WINDOWS_STORE)
    set_property(SOURCE ${SCRIPTS} ${BABYLONSCRIPTS} PROPERTY VS_DEPLOYMENT_CONTENT 1)
    set_property(SOURCE ${SCRIPTS} ${BABYLONSCRIPTS} PROPERTY VS_DEPLOYMENT_LOCATION "Scripts")
    
    set_property(SOURCE ${BABYLON_NATIVE_DEPENDENCIES} PROPERTY VS_DEPLOYMENT_CONTENT 1)
    set_property(SOURCE ${BABYLON_NATIVE_DEPENDENCIES} PROPERTY VS_DEPLOYMENT_LOCATION ".")
else()
    foreach(script ${SCRIPTS} ${BABYLONSCRIPTS})
        get_filename_component(SCRIPT_NAME "${script}" NAME)
        add_custom_command(
            OUTPUT "Scripts/${SCRIPT_NAME}"
            COMMAND "${CMAKE_COMMAND}" -E copy "${CMAKE_CURRENT_SOURCE_DIR}/${script}" "${CMAKE_CURRENT_BINARY_DIR}/Scripts/${SCRIPT_NAME}"
            COMMENT "Copying ${SCRIPT_NAME}"
            MAIN_DEPENDENCY "${CMAKE_CURRENT_SOURCE_DIR}/${script}")
    endforeach()
    
    foreach(dependency ${BABYLON_NATIVE_DEPENDENCIES})
        add_custom_command(TARGET Playground POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${dependency} $<TARGET_FILE_DIR:Playground>)
    endforeach(dependency)
endif()
