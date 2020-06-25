# Settings and Configuration

MuseScore 3 had one “Preferences” repository and a general list of keys. And it was used in all places of the application.
  
In MuseScore 4, we want to separate the entire application into modules, so we do not need to have one place with a list of all the settings keys. Each module must have its own settings.
And we also don’t want consumers to use the settings repository directly, because it’s more difficult to manage, we don’t know exactly what settings are used in the module and it’s difficult to make mocks for unit testing.  
Therefore, we expect that each module will have a configuration that will use a common settings repository.

* [Settings](#settings)
  * [Key](#key)
  * [Value](#value)
  * [Items](#items)
  * [Load](#load)
  * [Get value](#get-value)
  * [Set value](#set-value)
  * [Subscribe](#subscribe)
* [Module configuration](#module-configuration)
  * [Add configuration](#add-configuration)
  * [Using configuration](#using-configuration)

## Settings

Settings is the general settings repository, it is intended for use in module configurations.

### Key

The key consists of the module name and the key for the value.

```cpp
static const Settings::Key BACKGROUND_COLOR("notation_scene", "ui/canvas/background/color");
```

### Value

The type of value is determined automatically by the type of data it contains. Currently supported types are:

```cpp
struct Val
{
    enum Type {
        Undefined = 0,
        Bool,
        Int,
        Double,
        String,
        Color
    };
}


int val = settings()->value(key).toInt();

```

### Items

Each module can add a settings item to the repository, which will be used on the settings change form.

```cpp
settings()->addItem(Settings::Key("module_name", "key_name"), Settings::Val(QColor("#D6E0E9")));
```

Сan find an item by key or get all the items to use them on the settings change form

```cpp
const Settings::Item& item = settings()->findItem(Settings::Key("module_name", "key_name"));
const std::map<Settings::Key, Settings::Item>& items = settings()->items();

```

### Load

After all the settings items have been added, it is necessary to call the `load`, at this point the setting's value will be loaded from the disk and cached.

```cpp
settings()->load();
```

### Get value

Using the key, you can get the setting value. The setting value is resolved by such an algorithm (pseudocode)

```cpp
// algorithm
Settings::Val Settings::value(const Key& key) const
{
    Item item = findItem(key);
    if (item.isNull()) {
        return m_settings->value(key);
    }

    if (item.val.isNull()) {
        return item.defaultVal;
    }

    return item.val;
}

// get
int val = settings()->value(key).toInt();

```

Can also get the default value, only for those settings that were added as items earlier.

```cpp
QColor defColor = settings()->defaultValue(BACKGROUND_COLOR).toQColor();
```

### Set value

Сan set the value by key. When the value is set, a notification is sent that the value has changed.

```cpp
settings()->setValue(key, val);
```

### Subscribe

Сan subscribe to a value change notification

```cpp
settings()->valueChanged(BACKGROUND_COLOR).onReceive(nullptr, [this](const Settings::Val& val) {
    LOGD() << "BACKGROUND_COLOR changed: " << val.toString();
});
```

## Module configuration

We do not expect the settings repository (Settings) to be used directly. Instead, each module should have a configuration (interface and implementation) with methods for obtaining the settings used in this module.
This may seem redundant (over-engineering), but it reduces complexity (reduces entropy :)), adds order and clarity to the system. And also this approach will provide simpler unit testing. Having spent time writing the configuration (5 minutes) once, we will always use a simpler system, especially when each module will have a configuration file and we will always know where to look.

### Add configuration

Example configuration for view notation.

Add configuration interface

```cpp
class ISceneNotationConfiguration : MODULE_EXPORT_INTERFACE
{
    INTERFACE_ID(ISceneNotationConfigure)
public:
    virtual ~ISceneNotationConfiguration() = default;

    // get background color
    virtual QColor backgroundColor() const = 0;
    // get channel about background color change
    virtual async::Channel<QColor> backgroundColorChanged() = 0;

    // get selection proximity (not need notify about change)
    virtual int selectionProximity() const = 0;
};

```

Add implementation

Header

```cpp
class SceneNotationConfiguration : public ISceneNotationConfiguration
{
public:

    // method to call initialization (it is not in the interface)
    void init();

    // Interface implementations
    QColor backgroundColor() const override;
    async::Channel<QColor> backgroundColorChanged() override;

    int selectionProximity() const override;

private:
    // The channel that will be returned to the consumer
    // through which we will notify of the change.
    async::Channel<QColor> m_backgroundColorChanged;
};
```

Cpp

```cpp
// The name of the module (you can not declare a variable, just for convenience)
static std::string module_name("notation_scene");

// Keys for used settings
static const Settings::Key BACKGROUND_COLOR(module_name, "ui/canvas/background/color");
static const Settings::Key SELECTION_PROXIMITY(module_name, "ui/canvas/misc/selectionProximity");

void SceneNotationConfiguration::init()
{
    using Val = Settings::Val;

    // Add a setting item
    settings()->addItem(BACKGROUND_COLOR, Val(QColor("#D6E0E9")));

    // We subscribe to change the settings,
    // convert to the desired type and forward it.
    settings()->valueChanged(BACKGROUND_COLOR).onReceive(nullptr, [this](const Settings::Val& val) {
        LOGD() << "BACKGROUND_COLOR changed: " << val.toString();
        m_backgroundColorChanged.send(val.toQColor());
    });

    // Add other a setting item (not need notify about change)
    settings()->addItem(SELECTION_PROXIMITY, Val(6));
}

QColor SceneNotationConfiguration::backgroundColor() const
{
    // return value
    return settings()->value(BACKGROUND_COLOR).toQColor();
}

Channel<QColor> SceneNotationConfiguration::backgroundColorChanged()
{
    // return channel for notification about changed
    return m_backgroundColorChanged;
}

int SceneNotationConfiguration::selectionProximity() const
{
    // return value
    return settings()->value(SELECTION_PROXIMITY).toInt();
}
```

Configuration registration and initialization call

```cpp

static SceneNotationConfiguration* m_configuration = new SceneNotationConfiguration();

void NotationSceneModule::registerExports()
{
    framework::ioc()->registerExport<ISceneNotationConfiguration>(moduleName(), m_configuration);
}

void NotationSceneModule::onInit()
{
    m_configuration->init();
}

```

### Using configuration

Configuration Injection

```cpp
#include "modularity/ioc.h"
#include "iscenenotationconfiguration.h"
...

class NotationPaintView : public QQuickPaintedItem, public async::Asyncable
{
    INJECT(notation_scene, ISceneNotationConfiguration, configuration)
    ...

private:

    QColor m_backgroundColor;
}
```

Getting the value and subscribing to change it

```cpp
NotationPaintView::NotationPaintView()
    : QQuickPaintedItem()
{
    ...
    // configuration
    m_backgroundColor = configuration()->backgroundColor();
    configuration()->backgroundColorChanged().onReceive(this, [this](const QColor& c) {
        m_backgroundColor = c;
        update();
    });
```
