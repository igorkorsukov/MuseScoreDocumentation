# Channels and Notifications

* [Channels](#channels)
* [Notifications](#notifications)

In Qt there is a powerful system for asynchronous interaction - signals and slots.
But there are several problems with it:

* __Dependence on Qt__ (and we try to reduce the dependence on Qt as much as possible)
* It is not convenient to use with interfaces; in interfaces, you need to add the `virtual QObject* getQObject() = 0` method; and return a `QObject*`.
* It's harder to do mocks for unit tests.
* It is unsafe to use lambdas because there is no automatic disconnect for lambdas and there will be a crash if the receiver is deleted

Therefore, the use of Qt systems of signals and slots is not suitable for us.

At the same time, we plan that in MuseScore 4 there will be many asynchronous interactions. Accordingly, the system should be easy to use.

For these purposes, a number of primitives have been developed for simple asynchronous interaction. The channels in the GoLang language served as inspiration.

## Channels

Сhannels is a primitive for bidirectional transmission of typed data.

* [Can subscribe to the channel and send data](#can-subscribe-to-the-channel-and-send-data)
* [Channels can be returned from methods](#channels-can-be-returned-from-methods)
* [Channels can be passed to methods](#channels-can-be-passed-to-methods)
* [Can unsubscribe from the channel](#can-unsubscribe-from-the-channel)
* [Channel can be closed](#channel-can-be-closed])

### Can subscribe to the channel and send data

```cpp
Channel<int> ch;
ch.onReceive(nullptr, [](int val) {
    std::cout << "onReceive val: " << val << "\n";
});

ch.send(1); // will be printed: 'onReceive val: 1'

```

### Channels can be returned from methods

```cpp
struct Counter {
    Channel<int> m_ch;
    int m_val = 0;

    void increment()
    {
        ++m_val;
        m_ch.send(m_val);
    }

    Channel<int> channel() const
    {
        return m_ch;
    }
};


int main(int argc, char* argv[])
{
    Counter counter;

    counter.channel().onReceive(nullptr, [](int val) {
        std::cout << "counter: " << val << "\n";
    });

    for (int i = 0; i < 5; ++i) {
        counter.increment(); // will be printed: 'counter: n'
    }
}
```

### Channels can be passed to methods

If the subscription occurs inside the class method, then it is required to inherit from the `Asyncable` interface and you need to pass `this` to the subscription method so that there is auto disconnect when deleting an object

```cpp
class Receiver : public async::Asyncable
{
public:
    Receiver(const Channel<int>& ch)
        : m_ch(ch)
    {
        m_ch.onReceive(this, [](int val) {
            std::cout << "Receiver channel: " << val << "\n";
        });
    }

private:
    Channel<int> m_ch;
};

int main(int argc, char* argv[])
{
    Counter counter;
    Channel<int> ch = counter.channel();

    {
        Receiver r(ch);
        for (int i = 0; i < 5; ++i) {
            counter.increment(); // will be printed: 'Receiver channel: n'
        }
    } // When deleting an object, all its callbacks will be unsubscribed.
}

```

### Can unsubscribe from the channel

```cpp

class Receiver : public async::Asyncable
{
...
    void unsubscribe()
    {
        m_ch.resetOnReceive(this);
    }
}

int main(int argc, char* argv[])
{
    Counter counter;
    Channel<int> ch = counter.channel();

    Receiver r(ch);
    for (int i = 0; i < 5; ++i) {
        counter.increment(); // will be printed: 'Receiver channel: n'
    }

    r.unsubscribe();

    for (int i = 0; i < 5; ++i) {
        counter.increment(); // nothing will be printed
    }

}

```

### Channel can be closed

Closing the channel notifies subscribers that there will be no data coming from the channel.

```cpp
struct Pocessor
{
    Channel<int> m_ch;

    Channel<int> channel() const
    {
        return m_ch;
    }

    void process()
    {
        for (int task = 0; task < 5; ++task) {
            int val = doCalculate(task);
            m_ch.send(val);
        }

        m_ch.close();
    }
};

struct Saver : public async::Asyncable
{
    Stream m_stream;
    void saveFromChannel(const Channel<int>& ch)
    {
        m_stream.open();
        ch.onReceive(this, [this](int val) {
            m_stream.write(val);
        });

        ch.onClose(this, [this]() {
            m_stream.close();
        });
    }
};

int main(int argc, char* argv[])
{
    Pocessor p;
    Saver s;

    s.saveFromChannel(p.channel());

    p.process();
}
```

## Notifications

Notification is designed to send notifications.
They have all the qualities of channels, they are only used without arguments and have slightly different semantics.

```cpp
// Notification sender
class Selector
{
public:
    void select(int val)
    {
        m_selection = val;
        m_selecttionChanged.notify();
    }

    int selection() const
    {
        return m_selection;
    }

    Notification selecttionChanged() const
    {
        return m_selecttionChanged;
    }

private:
    int m_selection = 0;
    Notification m_selecttionChanged;
};

// Notification receiver
class SelectionPrinter : public async::Asyncable
{
    void setSelector(Selector* s)
    {
        if (m_selector) {
            m_selector->selecttionChanged().resetOnNotify(this); // unsubscribe from previous
        }

        m_selector = s;
        m_selector->selecttionChanged().onNotify(this, [this]() {
            std::cout << "Selection changed, new selection: " << m_selector->selection() << "\n";
        });
    }

private:
    Selector* m_selector = nullptr;
};


int main(int argc, char* argv[])
{
    Selector* sel = new Selector();
    SelectionPrinter* p = new SelectionPrinter();

    p->setSelector(sel);

    sel->select(5); // will be printed: 'Selection changed, new selection: 5'
}

```
