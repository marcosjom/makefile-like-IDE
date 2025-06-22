
# From sys-nbframework-src

The content of this repository is part of [sys-nbframework-src](https://github.com/marcosjom/sys-nbframework-src); it exists to expose and document a specific feature of the bigger framework.

# makefile-like-IDE

`MakefileFuncs.mk` contains functions that allows you to describe a workspace similarly to XCode, Visual-Studio and other IDEs and automatizes the compilation of your targets.

# Origin

Created by [Marcos Ortega](https://mortegam.com/) to facilitate the compilation of projects on Linux, Android and MacOS by just keeping the description of each project synced with their IDE counterparts.

# Features

- Unified way of describing your projects; like in your IDEs.
- Command line compilation.
- Linux, Android, MacOS and other operating systems that supports compiling with `make`. 

# Structure

Its purpose is to describe your project in a structure that allows you to set individual compiler-flags to each code-group. The structure is:

- `Workspace`
  - `Project`
    - `Target` (exe, lib, so, dlyb)
      - `Code-group`

The `workspace` is the folder you are explicitly calling `make` from. You can import your current project and dependencies from other folders.

Once all the `Projects`, `Targets` and `Code-groups` were imported, the current call to `make` will build all (or a specific) target; compiling each file with their respective flags, merging outputs and linking libraries.

Basically is a command-line version of how you manage and build your projects, inspired in how you organize your work in XCode.

The `compiler` and `tool-chain` is selected automatically or explicitly by the `NB_CFG_HOST=` param. As example `NB_CFG_HOST=Android` will compile using `Android NDK` tool-chain.

Check the `MakefileProject.mk` and `Makefile` for implementation examples; those files exists in most of my repositories.

# Example

For each of your workspaces create a `MakefileProject.mk` file to describe its projects:

```
#-------------------------
# PROJECT 1
#-------------------------

$(eval $(call nbCall,nbInitProject))

NB_PROJECT_NAME             := my_project
NB_PROJECT_CFLAGS           :=
NB_PROJECT_CXXFLAGS         :=
#Add debug specific
ifeq ($(NB_WORKSPACE_BLD_CFG),debug)
   NB_PROJECT_CFLAGS        += -DDEBUG=1
   NB_PROJECT_CXXFLAGS      += -DDEBUG=1
endif
NB_PROJECT_INCLUDES         := include
#only if NB_LIB_Z_SYSTEM is not set
ifeq ($(NB_LIB_Z_SYSTEM),)
   NB_PROJECT_CFLAGS        += -DZLIB_DEBUG=1
   NB_PROJECT_CXXFLAGS      += -DZLIB_DEBUG=1
endif

#-------------------------
# PROJECT 1, TARGET 1
#-------------------------

$(eval $(call nbCall,nbInitTarget))

NB_TARGET_NAME              := my_lib_static
NB_TARGET_PREFIX            := lib
NB_TARGET_SUFIX             := .a
NB_TARGET_TYPE              := static
    
#-------------------------
# PROJECT 1, TARGET 1, CODE GROUP 1
#-------------------------

$(eval $(call nbCall,nbInitCodeGrp))

NB_CODE_GRP_NAME            := my_static_lib_src
NB_CODE_GRP_FLAGS_REQUIRED  += MY_LIB_CORE_STATIC
NB_CODE_GRP_CFLAGS          += 
NB_CODE_GRP_CXXFLAGS        += -std=c++20
NB_CODE_GRP_INCLUDES        += path/include path2/include
NB_CODE_GRP_SRCS            := path/src/my_src.cpp
NB_CODE_GRP_LIBS            += pthread m

$(eval $(call nbCall,nbBuildCodeGrpRules))

#-------------------------
# CREATE TARGET 1 RULES
#-------------------------

$(eval $(call nbCall,nbBuildTargetRules))

#-------------------------
# PROJECT 1, TARGET 2
#-------------------------

$(eval $(call nbCall,nbInitTarget))

NB_TARGET_NAME              := my_exe
NB_TARGET_PREFIX            := 
NB_TARGET_SUFIX             := .exe
NB_TARGET_TYPE              := exe
    
#-------------------------
# PROJECT 1, TARGET 2, CODE GROUP 1
#-------------------------

$(eval $(call nbCall,nbInitCodeGrp))

NB_CODE_GRP_NAME            := my_exe_src
NB_CODE_GRP_FLAGS_ENABLES   += MY_LIB_CORE_STATIC
NB_CODE_GRP_CFLAGS          += 
NB_CODE_GRP_CXXFLAGS        += -std=c++20
NB_CODE_GRP_INCLUDES        += path/include path2/include
NB_CODE_GRP_SRCS            := path/src/main.cpp
NB_CODE_GRP_LIBS            += 

$(eval $(call nbCall,nbBuildCodeGrpRules))

#-------------------------
# CREATE TARGET 2 RULES
#-------------------------

$(eval $(call nbCall,nbBuildTargetRules))

#-------------------------
# CREATE PROJECT 1 RULES
#-------------------------

$(eval $(call nbCall,nbBuildProjectRules))

```

In this example your workspace contains a project with two targets: `my_lib_static` and `my_exe`; each one with a list of source-files. And `my_exe` has been set to depend of `my_lib_static` by enabling your `MY_LIB_CORE_STATIC` flag (you can name the dependencies flags as you like).

In your `Makefile` you can import this worskpace's projects, and other projects:

```
#Configure
NB_CFG_PRINT_INTERNALS := 0
NB_CFG_PRINT_INFO      := 0

#Import functions
include MakefileFuncs.mk

#Init workspace
$(eval $(call nbCall,nbInitWorkspace))

#Local projects
include MakefileProject.mk

#Other dependencies projects
#include ../other/MakefileProject.mk
#include ../other2/MakefileProject.mk

#Build workspace
$(eval $(call nbCall,nbBuildWorkspaceRules))
```

With this example you now can call:

```
make my_lib_static
```

or 

```
make my_exe
```

The first command will build only `my_lib_static`. The second command will build `my_exe`, but as `my_lib_static` is a dependency, it will also be built and linked to `my_exe`.

# Debugging

Before importing your `MakefileProject.mk` files, set these variables to `1` or `0`:

- `NB_CFG_PRINT_INTERNALS`, will output the `make` rules automatically generated by your projects.
- `NB_CFG_PRINT_INFO`, will output descriptive messages.

# Contact

Visit [mortegam.com](https://mortegam.com/) for more information and visual examples of projects built with this libray.

May you be surrounded by passionate and curious people. :-)
