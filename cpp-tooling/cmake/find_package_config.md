# Writing Config Files for Package Import

Date: 29 April 2026

The goal of a `<packageName>Config.cmake` file is to import externally-defined
targets into your CMake project. These configuration files provide a set of imported
libraries and executables to make available to other projects.

## Useful Variables

- `CMAKE_FIND_PACKAGE_NAME`
- `${CMAKE_FIND_PACKAGE_NAME}_FIND_REQUIRED`
- `${CMAKE_FIND_PACKAGE_NAME}_FIND_QUIETLY`
- `${CMAKE_FIND_PACKAGE_NAME}_FIND_COMPONENTS`: All components specified by the
  `find_package()` call.
- `${CMAKE_FIND_PACKAGE_NAME}_FIND_REQUIRED_<component>`: Query whether the
  specified component was specified as required.
- `${CMAKE_FIND_PACKAGE_NAME}_NOT_FOUND_MESSAGE`: Used to report errors during
  the find operation.
- `${CMAKE_FIND_PACKAGE_NAME}_FOUND`: Determine if the requested package has been found.
  - This should default to false until the entire find operation is complete,
    components and all. 

## Guidelines

1. Per Scott Craig, don't use `message()` in a config file.
  - If `message( FATAL_ERROR ... )` were used, the package could never be treated as
    optional.
0. Verify that all required components can be found *before* creating any
   imported targets. This ensures that either all targets are imported or none.
0. Consider generating the set of all component names as part of the creation of
   this file.
0. Not sure if this is required or not, but I've seen config files use
   underscore-prefixed local variables. This is likely done to prevent variable
   collisions with the project importing your package.
  
## Steps

1. Determine the set of valid importable targets and their analogous component names
   if they differ. 
  - **NOTE**: The list of target names and the list of component names should be
    ordered such that `targetNames[i]` pairs with `componentNames[i]`.
0. Copy the list of requested components to a local variable. If no components are specified
   (${CMAKE_FIND_PACKAGE_NAME}_FIND_COMPONENTS is empty), set the local variable
   to the set of all component names defined in the previous step.
0. Iterate through the list of requested components, checking to see if the
   given component is in the list of valid components.
   - As these components are found in the list of valid components, save the
     index of the component relative to the list of valid_components. This is
     needed to map the component name to the target name.
0. Iterate through the component list again, this time calling `include()` to
   include the relevant `<targetName>Config.cmake` for that target.
   - Save the imported target names to another list variable. 
0. Add an interface target that links to all of the imported targets (use the
   variable defined in the previous step). This allows the user to simply link
   to all components imported in one line. 
