# Modularity

## Software’s Primary Technical Imperative: Managing Complexity

> Managing complexity is the most important technical topic in software development.
In my view, it’s so important that Software’s Primary Technical Imperative has to be
managing complexity.  
Dijkstra pointed out that no one’s skull is really big enough to contain a modern computer program (Dijkstra 1972), which means that we as software developers
shouldn’t try to cram whole programs into our skulls at once; we should try to organize our programs in such a way that we can safely focus on one part of it at a time.
The goal is to minimize the amount of a program you have to think about at any one
time. You might think of this as mental juggling—the more mental balls the program
requires you to keep in the air at once, the more likely you’ll drop one of the balls,
leading to a design or coding error.  
At the software-architecture level, the complexity of a problem is reduced by dividing
the system into subsystems. Humans have an easier time comprehending several simple pieces of information than one complicated piece. The goal of all software-design
techniques is to break a complicated problem into simple pieces. The more independent the subsystems are, the more you make it safe to focus on one bit of complexity
at a time. Carefully defined objects separate concerns so that you can focus on one
thing at a time. Packages provide the same benefit at a higher level of aggregation.  
Keeping routines short helps reduce your mental workload. Writing programs in
terms of the problem domain, rather than in terms of low-level implementation
details, and working at the highest level of abstraction reduce the load on your brain.  
The bottom line is that programmers who compensate for inherent human limitations write code that’s easier for themselves and others to understand and that has
fewer errors.

(c) Steven C. McConnell "Code Complete" Chapter 5

## Modules

We divide the entire application into a modules.
A module is a functional unit that, on the one hand, contains independent value, on the other hand, an application can work without this module.

### Structure

Use this files structure

root/

* CMakeList.txt - module cmake project
* {name}module.cpp/h - module setup
* isomeinterface.h - public interfaces
* {name}types.h - public types
* {name}errors.h - public error codes
* internal/ - dir with private implementations
* view/ - dir with classes used into view (qml) or widgets
* qml/MuseScore/{Name} - dir with qml files

### Project

There is a module project template `${PROJECT_SOURCE_DIR}/build/module.cmake`. To use it, you need to set the necessary variables and then make the template include.

Example:

```cpp
set(MODULE scores)

set(MODULE_QRC scores.qrc)

set(MODULE_QML_IMPORT ${CMAKE_CURRENT_LIST_DIR}/qml )

set(MODULE_SRC
    ${CMAKE_CURRENT_LIST_DIR}/scoresmodule.cpp
    ${CMAKE_CURRENT_LIST_DIR}/scoresmodule.h
    ${CMAKE_CURRENT_LIST_DIR}/iopenscorecontroller.h
    ${CMAKE_CURRENT_LIST_DIR}/view/scoresmodel.cpp
    ${CMAKE_CURRENT_LIST_DIR}/view/scoresmodel.h
    ${CMAKE_CURRENT_LIST_DIR}/internal/openscorecontroller.cpp
    ${CMAKE_CURRENT_LIST_DIR}/internal/openscorecontroller.h
    )

include(${PROJECT_SOURCE_DIR}/build/module.cmake)
```

Possible variables see in `module.cmake`

### Module setup

Each module must have a setup that implements the `IModuleSetup` interface.
You can implement not all methods, but only the necessary ones, the required method is only the name of the module.

```cpp
#include "modularity/imodulesetup.h"

namespace mu {
namespace scores {
class ScoresModule : public framework::IModuleSetup
{
public:

    std::string moduleName() const override;
    void registerExports() override;
    void registerResources() override;
    void registerUiTypes() override;
    void onInit() override;
};
}
}
```

Possible methods see in `imodulesetup.h`, also see examples of setups of other modules.

The module setup must be added to the list of modules in `main/modulessetup.cpp`  
And module must be added to list of linking in `main/CMakeLists.txt`

### Public interfaces

The public interfaces of the module, those that can be used in other modules, must be located in the root of the module. This makes it easy to use and understand what is for public use and what is not.
  
Public interfaces can be exported through the dependency injection system, then they must be "exportable" and have an interface identifier (see [Dependency Injection](DependencyInjection.md)). And can be simply returned from other public interfaces.

If the module has interfaces that are intended to be used only inside the module, then they must be placed in the `internal` directory.

### Public types

All public types that can be used in other modules must be in the `{name}types.h` file.

It is assumed that public types are simple structures, enumerators, usings, etc., without implementation files, so that we do not need to link the module to use them.

Types that are supposed to be used only inside the module should be in the `internal` directory.

### Error codes

Public error codes must be in the `{name}errors.h` file, see [Error handling](ErrorHandling.md)

### Internal directory

The `internal` directory contains the implementation of interfaces, services, logic, etc., can be structured into subfolders, if necessary.

### View directory

The `view` directory contains Qml models and types, and can also contain a dialogues widgets, if necessary, all that is related to view.

### Qml

The qml/MuseScore/{Name} directory contains `qmldir` file and qml files for view.
