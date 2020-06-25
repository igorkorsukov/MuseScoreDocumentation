# Actions

`Actions` are intentions that are sent from the view to the model. The model processes the action, changes its state and sends a notification of changes. See [interact workwlow](InteractWorkflow.md).

* [Action](#action)
* [Dispatcher](#dispatcher)

## Action

The action consists of a name and a title (_other properties may be added in the future_)

```cpp
struct Action {
    ActionName name;
    std::string title;

    bool isValid() const { return !name.empty(); }
};
```

Each module has its own list of actions; there is no one global place with all actions.  
The name of the action consists of the namespace and the name of the action itself.  
Actions can be without arguments (most) and with arguments. For actions with arguments, you need to write a list of arguments in the comment - their order, type, names.  
For example, actions processed by notation are in the file `/domain/notation/notationactions.cpp`

```cpp
//! NOTE Only actions processed by notation

static const std::vector<Action> m_actions = {
    {
        "domain/notation/note-input",
        QT_TRANSLATE_NOOP("action", "Note Input")
    },
    {
        "domain/notation/pad-note-4",
        QT_TRANSLATE_NOOP("action", "Quarter Note")
    },
    {
        "domain/notation/pad-note-8",
        QT_TRANSLATE_NOOP("action", "8th Note")
    },
    ...
    {
        "domain/notation/put-note", // args: QPoint pos, bool replace, bool insert
        QT_TRANSLATE_NOOP("action", "Put Note")
    }
    ...
};

```

## Dispatcher

To send actions, use the `Action Dispatcher`.
At the moment, you can use the dispatcher only in C++ code and we do not plan to add the ability to use the dispatcher in Qml directly.

Adding a dispatcher dependency:

```cpp
...
#include "modularity/ioc.h"
#include "actions/iactionsdispatcher.h"
...
class NotationToolBarModel
{
    INJECT(notation_scene, actions::IActionsDispatcher, dispatcher)
    ...
}
```

Submit Actions

```cpp
// without arguments
dispatcher()->dispatch("domain/notation/note-input");

// with arguments
dispatcher()->dispatch("domain/notation/put-note", ActionData::make_arg3<QPoint, bool, bool>(logicPos, replace, insert));
```

Handler Registration

```cpp

// without arguments (like Qt connect)
dispatcher()->reg("domain/notation/note-input", this, &NotationActionController::toggleNoteInput);

dispatcher()->reg("domain/notation/pad-note-4", [this]() { padNote(Pad::NOTE4); });

// with argumens
dispatcher()->reg("domain/notation/put-note", this, &NotationActionController::putNote);

// Where
void NotationActionController::putNote(const actions::ActionData& data)
{
    ...
    QPoint pos = data.arg<QPoint>(0);
    bool replace = data.arg<bool>(1);
    bool insert = data.arg<bool>(2);

    notation->putNote(pos, replace, insert);
}

```
