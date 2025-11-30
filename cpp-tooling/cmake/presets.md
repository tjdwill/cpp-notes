# CMake Presets

Docs: https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html

CMake allows users to specify presets via JSON files. There are two types of
CMake preset files. The first file, `CMakePresets.json`, stores
project-wide build configurations. It is intended to be version-controlled,
allowing all developers to have the same basic configuration. The second file,
`CMakeUserPresets.json` is a user-specific preset file that stores configuration
data specific to a given user's machine. As such, it is *not* to be
version-controlled.

Here are some example files:


```jsonc
// `CMakePresets.json`
{
  "version": 10,
  "configurePresets": [
    {
      "name": "default",
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": {
          "type": "BOOL",
          "value": "ON"
        }
      }
    },
    {
      "$comment": "A vcpkg-specific preset.",
      "name": "vcpkg",
      "inherits": "default",
      "cacheVariables": {
        "CMAKE_TOOLCHAIN_FILE": {
          "type": "PATH",
          "value": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
        }
      }
    }
  ]
}
//-----------------------------------------------------------------------------
// CMakeUserPresets.json
// Specify specific user configurations
{
  "version": 10,
  "configurePresets": [
    {
      "name": "debug",
      "inherits": "default",
      "binaryDir": "${sourceDir}/build_Debug",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": {
          "type": "STRING",
          "value": "Debug"
        },
        "CMAKE_CXX_COMPILER": "clang++-20"
      },
      "environment":
      {
        "LOCAL_INCLUDES": "$env{HOME}/.local/include",
        "CMAKE_PREFIX_PATH": "$env{LOCAL_INCLUDES}/bde${pathListSep}$env{LOCAL_INCLUDES}/Qt/6.5.2/gcc_64${pathListSep}$penv{CMAKE_PREFIX_PATH}"
      }
    },
    {
      "name": "release",
      "inherits": "debug",
      "binaryDir": "${sourceDir}/build_Release",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Release"
      }
    }
  ]
}
```

One thing I had to learn was that `CMAKE_PREFIX_PATH` is to be specified as an
environment variable instead of a cache variable. Otherwise, CMake won't be able
to find the necessary CMake command files associated with packages loaded via
`find_package()`.