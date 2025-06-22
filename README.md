
# from sys-nbframework-src

The content of this repository is part of [sys-nbframework-src](https://github.com/marcosjom/sys-nbframework-src); it exists to expose and document specific feature of the bigger framework.

# makefile-like-IDE

`MakefileFuncs.mk` containing functions that allows you to describe a workspace similarly to XCode, Visual-Studio and other IDEs.

Its purpose is to describe your project in an structure and be able to set individual compiler-flags to each code-group. The structure is:

- `Workspace`
  - `Project`
    - `Target` (exe, lib, so, dlyb)
      - `Code-group`

The `workspace` is the folder you are explicitly calling `make` from. You can import your current project and dependencies from other folders.

Once all the `Projects`, `Targets` and `Code-groups` were imported, the current call to `make` will build all (or a specific) target; compiling each file with their respective flags, merging outputs and linking libraries.

Basically is a command-line version of how you manage and build your projects, inspired in how you organize your work in XCode.

The `compiler` and `tool-chain` is selected automatically or explicitly by the `NB_CFG_HOST=` param. As example `NB_CFG_HOST=Android` will compile using `Android NDK` tool-chain.

Check the `MakefileProject.mk` and `Makefile` for implementation examples; those files exists in most of my repositories.


# Origin

Created by [Marcos Ortega](https://mortegam.com/) to facilitate the compilation of projects on Linux, Android and MacOS by just keeping the description of each project synced with their IDE counterparts.

# Example

For each of your workspaces create a `MakefileProject.mk` file to describe its projects:

```
#-------------------------
# PROJECT
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
# TARGET 1
#-------------------------

$(eval $(call nbCall,nbInitTarget))

NB_TARGET_NAME              := my_lib_static
NB_TARGET_PREFIX            := lib
NB_TARGET_SUFIX             := .a
NB_TARGET_TYPE              := static
    
#-------------------------
# TARGET 1, CODE GROUP 1
#-------------------------

$(eval $(call nbCall,nbInitCodeGrp))

NB_CODE_GRP_NAME            := my_static_lib_src
NB_CODE_GRP_FLAGS_REQUIRED  += NB_MY_LIB_CORE_STATIC
NB_CODE_GRP_CFLAGS          += 
NB_CODE_GRP_CXXFLAGS        += -std=c++20
NB_CODE_GRP_INCLUDES        += path/include path2/include
NB_CODE_GRP_SRCS            := path/src/my_src.cpp
NB_CODE_GRP_LIBS            += pthread m

$(eval $(call nbCall,nbBuildCodeGrpRules))

#-------------------------
# TARGET RULES
#-------------------------

$(eval $(call nbCall,nbBuildTargetRules))

#-------------------------
# TARGET 2
#-------------------------

$(eval $(call nbCall,nbInitTarget))

NB_TARGET_NAME              := my_exe
NB_TARGET_PREFIX            := 
NB_TARGET_SUFIX             := .exe
NB_TARGET_TYPE              := exe
    
#-------------------------
# TARGET 2, CODE GROUP 1
#-------------------------

$(eval $(call nbCall,nbInitCodeGrp))

NB_CODE_GRP_NAME            := my_exe_src
NB_CODE_GRP_FLAGS_ENABLES   += NB_MY_LIB_CORE_STATIC
NB_CODE_GRP_CFLAGS          += 
NB_CODE_GRP_CXXFLAGS        += -std=c++20
NB_CODE_GRP_INCLUDES        += path/include path2/include
NB_CODE_GRP_SRCS            := path/src/main.cpp
NB_CODE_GRP_LIBS            += 

$(eval $(call nbCall,nbBuildCodeGrpRules))

#-------------------------
# TARGET RULES
#-------------------------

$(eval $(call nbCall,nbBuildTargetRules))

#-------------------------
# PROJECT RULES
#-------------------------

$(eval $(call nbCall,nbBuildProjectRules))

```

In this example your workspace contains a project with two targets: `my_lib_static` and `my_exe`; each one with its source files listed. And `my_exe` has been set to depend of `my_lib_static` by enabling your `NB_MY_LIB_CORE_STATIC` flag (you can name the dependencies flags as you like).

In your `Makefile` you can import this worskpace's' projects, and other projects:

```
#Import functions
include MakefileFuncs.mk

#Init workspace
$(eval $(call nbCall,nbInitWorkspace))

#Local project
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

# Contact

Visit [mortegam.com](https://mortegam.com/) for more information and visual examples of projects built with this libray.

May you be surrounded by passionate and curious people. :-)
