# vcpkg with CMake

[Source](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-bash)

Assuming we have `vcpkg` installed and the relevant environment variables set:

1. Create the project directory
2. Create a `vcpkg.json` manifest file.
    
    ```bash
    $ cd project
    # ... now in project dir
    $ vcpkg new --application
    ```
Add dependencies to the file.

3. Setup `CMakePresets.json` and `CMakeUserPresets.json`

CMakePresets.json

    ```json
    {
      "version": 2,
      "configurePresets": [
        {
          "name": "vcpkg",
          "generator": "Ninja",
          "binaryDir": "${sourceDir}/build",
          "cacheVariables": {
            "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
          }
        }
      ]
    }
    ```

CMakeUserPresets.json (leave out of version control):

    ```json
    {
        "version": 2,
        "configurePresets": [
          {
            "name": "default",
            "inherits": "vcpkg",
            "environment": {
              "VCPKG_ROOT": "<path to vcpkg>"
            }
          }
        ]
    }
  ```


## References

- [vcpkg in CMake projects](https://learn.microsoft.com/en-us/vcpkg/users/buildsystems/cmake-integration)
- [vcpkg install](https://learn.microsoft.com/en-us/vcpkg/commands/install)
