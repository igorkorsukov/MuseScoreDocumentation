# Launcher and user interaction

* [Launcher](#launcher)
  * [Open](#open)
  * [Uri](#uri)
  * [Params](#params)
  * [Mode](#mode)
  * [Return code](#return-code)
  * [Example](#example)
* [Interactive](#interactive)
  * [Questions](#questions)
  * [Messages](#messages)
  * [Select file](#select-file)
  * [Require](#require)

## Launcher

This is service for opening pages and dialogues.  
It is available like other services through DI in `cpp` and through `api` in `qml`.

### Open

To open a page or dialog, you need to do

```cpp

launcher()->open("musescore://some/path");

```

or in `Qml`

```cpp

api.launcher.open("musescore://some/path");

```

where `musescore://some/path` is `URI`  
  
### Uri

Each page or dialog has a `URI`, for example:

* `"musescore://home"` - Home page
* `"musescore://notation"` - Notation page
* `"musescore://sequencer"` - Sequencer page
* `"musescore://publish"` - Publish page
* `"musescore://devtools/launcher/sample"` - test dialogue for demonstration
* etc

The mapping between `URI` and implementation is in `LaunchResolver.qml`, that is, to add a new page or dialogue that could be opened using the launcher, you need to add the appropriate mapping.

### Params

You can pass parameters to opened pages and dialogs, if they have them.  
Parameter pattern: `musescore://some/path?param1=value1&paramn=valuen`  

For example:  

```cpp
launcher()->open("musescore://devtools/launcher/sample?title=Test&color=#0F9D58");
```

The name of the parameters must correspond to the name of the properties of `Qml` dialogs, or have the appropriate mapping in `LaunchResolver.qml`.

### Mode

Dialogues can be shown modally, for this you need to add the `modal=true` parameter in `URI`.  

Dialogues can be showed synchronously or asynchronously; for synchronous showing, need to add the `sync=true` parameter.

### Return code

Dialogues can return an error code and value, they can be written to the `ret` property

### Example

Qml file `DevTools/Launcher/SampleDialog.qml`

```cpp
import QtQuick 2.7
import MuseScore.Ui 1.0
import MuseScore.UiComponents 1.0

QmlDialog {

    id: root

    property var color: "#444444"

    width: 400
    height: 400

    Rectangle {

        anchors.fill: parent
        color: root.color

        TextInputField {
            id: input
            property var value: ""
            anchors.centerIn: parent
            width: 150
            height: 32
            onCurrentTextEdited: input.value = newTextValue
        }

        Row {
            anchors.bottom: parent.bottom
            anchors.right: parent.right
            anchors.rightMargin: 16
            height: 40
            width:  100
            spacing: 20

            FlatButton {

                width: 40
                text: "Ok"
                onClicked: {
                    root.ret = {errcode: 0, value: input.value }
                    root.hide()
                }
            }

            FlatButton {
                width: 40
                text: "Cancel"
                onClicked: {
                    root.ret = {errcode: 3 }
                    root.hide()
                }
            }
        }
    }
}

```

Need to add to `LaunchResolver.qml`

```cpp
r["musescore://devtools/launcher/sample"] = function(d) {
    return {path: "DevTools/Launcher/SampleDialog.qml", params: d.params}
};

```

Opening

`cpp`

```cpp
launcher()->open("musescore://devtools/launcher/sample?color=#0F9D58");
```

`qml`

```cpp
api.launcher.open("musescore://devtools/launcher/sample?color=#0F9D58");
```

## Interactive

This is service for user interaction, ask the user something, or inform something, so that the user reacts to it.  
It is available like other services through DI in `cpp`, but not available in `qml`.
In `qml`, it is unavailable intentionally, so that `qml` remains for view and logic does not leak into it.  

Do not use QMessageBox or other QWidgets dialogues, use only `Interactive`.
This is necessary for at least two reasons:

* To have to replace the implementation of the dialogues themselves and the style (in the future there will be only Qml dialogues)
* To be able to write unit tests, interactive can be replaced in mock.

If there is no method in `interactive` to display the desired dialogue, add the method to the `IInteractive` interface and implement it in `Interactive`, even using standard QWidget dialogues, which can be replaced later.

### Questions

You can ask the user a question using the standard buttons

```cpp
IInteractive::Button btn = interactive()->question("Test", "It works?", {
    IInteractive::Button::Yes,
    IInteractive::Button::No });

if (btn == IInteractive::Button::Yes) {
    LOGI() << "Yes!!";
} else {
    LOGI() << "No!!";
}
```

You can ask the user a question using custom buttons

```cpp
    int maybeBtn = int(IInteractive::Button::CustomButton) + 1;
    int btn = interactive()->question("Test", "It works?",{
        IInteractive::ButtonData(maybeBtn, "Maybe"),
        interactive()->buttonData(IInteractive::Button::No)
    });

    if (btn == maybeBtn) {
        LOGI() << "Maybe!!";
    } else {
        LOGI() << "No!!";
    }
```

### Messages

You can show the user a message (informing, warning or error)

```cpp
interactive()->message(IInteractive::Type::Critical, "Test", "This is critical text");
```

### Select file

You may be asked to select a file to open

```cpp
io::path filePath = interactive()->selectOpeningFile("Title", dir, filter);
```

### Require

You can do anything with the user using your dialogue  
(just like when using the launcher)

```cpp
RetVal<Val> rv = interactive()->require("musescore://devtools/launcher/sample?title='Test'");
if (rv.ret) {
    LOGI() << "received: " << rv.val.toString();
} else if (check_ret(rv.ret, Ret::Code::Cancel)) {
    LOGI() << "was cancelled";
} else {
    LOGE() << "some error: " << rv.ret.code();
}
```
