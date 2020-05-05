+++
title = 'QML Online - First stable version!'
[taxonomies]
tags = [ "qml", "qt", "webasm", "web", "internet", "online" ]
[extra]
header = "/assets/qmlonline-first-version/banner.png"
+++

Finally, after working since October and learning a bunch about [WebAssembly](https://webassembly.org/), [CSS](https://en.wikipedia.org/wiki/Cascading_Style_Sheets), [HTML](https://wikipedia.org/wiki/HTML) (sad, right ?) and [emscripten](https://emscripten.org/), I can happily announce a stable version of [qmlonline](https://patrickelectric.work/qmlonline/)!
In this post, I'm going to show the idea behind the project and some code that may help you with your future adventures.

<!-- more -->

# The initial steps

Everything starts with [QHot](https://github.com/patrickelectric/qhot), that I describe as _"Hot reload for nested QML files"_, a useful tool for anyone that likes to prototype UI elements or ideas with a real-time feedback of what you are typing in QML. I noticed that compiling the project or recalling qml/qmlscene tools just to test and check my ideas was pretty annoying and time-consuming, the desire to have something like [godbolt](https://godbolt.org/) or [quick-bench](http://quick-bench.com/) started growing. My objective was something that was closer to these tools but for QML development, and that is how **QHot** was born.

![image](https://raw.githubusercontent.com/patrickelectric/qhot/master/doc/example.gif)

With QHot working, I started to add some small features in the command line interface to have mostly of the functionalities that exist in qml/qmlscene, at least the most important ones for my use.


After some days, the idea of **QHot** working in the browser via webassembly started to grow, and with that, the initial work that would result in **qmlonline**.


# Journey

After some tweaks around **QHot**, it was possible to have the first version of qmlonline working.
The initial version was entirely made with QML, without HTML components, the editor was a [TextEdit](https://doc.qt.io/qt-5/qml-qtquick-textedit.html) with a fancy [QSyntaxHighlighter](https://doc.qt.io/qt-5/qsyntaxhighlighter.html). The text inside the **TextEdit** was used to create a new component with [Qt.createQmlObject](https://doc.qt.io/qt-5/qml-qtqml-qt.html#createQmlObject-method).

![image](/assets/qmlonline-first-version/full-qml.png)

With the initial version working, I started to move the interface to HTML, this was necessary to have a better shortcut handling system and a better integration with the browser for user inputs.

The code evolved from a simple **TextEdit** with some controls to a full HTML interface, for that, functions had to be imlpemented to help with the webassembly code to be accessible from the webpage.

From the beginning, I had an initial singleton class called **Util** that was conceived to be some kind of helper class for the QML code. This same class was used to create the interface between the JS and the webassembly via emscripten.

In general, two functions were created, `std::string Util::codeEMS() const` that returns the code that is being used in the QML to render the user component, and `void Util::setCodeEMS(const std::string& code)` that sets the code that should be rendered.
To access both functions and the class, [EMSCRIPTEN_BINDINGS](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html) were used.

```cpp
#include <emscripten/bind.h>

EMSCRIPTEN_BINDINGS(util) {
    emscripten::class_<Util>("Util")
        .function("code", &Util::codeEMS)
        .function("setCode", &Util::setCodeEMS);
    emscripten::function("self", &Util::self, emscripten::allow_raw_pointers());
}
```

That's probably the most important piece of code to do the integration between the C++ and JS.

And for the QML, well, you can check all QML here:

```js
...
import Util 1.0

ApplicationWindow {
    id: window
    title: "qmlonline"
    visible: true

    Connections {
        target: Util
        onCodeChanged: userParentItem.create(Util.code)
    }

    Item {
        id: userParentItem
        anchors.fill: parent
        property var userItem: null

        function create(textComponent) {
            if(userItem) {
                userItem.destroy()
            }
            userItem = Qt.createQmlObject(textComponent, userParentItem, "userItem")
        }

        Component.onCompleted: userParentItem.create(Util.code)
    }
}
```

Oh, [you can check the source code here](https://github.com/patrickelectric/qmlonline/blob/71342f68f5f99dcf9c2b69051b63ea2e83010ca8/src/util.cpp).

To build the project, I changed my approach a couple of times.
For the first test version, I did my development based in the official documents for [Qt webassembly](https://doc.qt.io/qt-5/wasm.html). But, compiling Qt for each machine that I use was a bit of a pain, so I started to use a docker available from this [Qt blog post](https://www.qt.io/blog/2019/03/05/using-docker-test-qt-webassembly).
And everything was great until newer versions of Qt and emscript were released, and after some time searching, I found this great [repository/developer](https://hub.docker.com/r/madmanfred/qt-webassembly/tags) with a couple of Qt containers for webassembly and different emscript versions.

And after building the project, you can always test it with `python3 -m http.server`.

# Where I am

![image](/assets/qmlonline-first-version/final-version.gif)

As you can see from the gif, the basic functionality is still the same from the original version, that was full QML.

The user experience has improved, the interface is now much smoother compared to the full QML version. Qt webassembly is great, but from my tests, the performance was not good and the browser/system integration needs improvement to have the same functionality as a normal webpage or application.

Again, remember to check [qmlonline](https://patrickelectric.work/qmlonline/) in your browser :)

If you have good QML examples and wish to add those, contact me via email, or create an issue in the [repository](https://github.com/patrickelectric/qmlonline), or send a PR, [it's really simple](https://github.com/patrickelectric/qmlonline/tree/gh-pages/qml/examples).

# What I'm working on

There is a couple of things that I'm still working to improve qmlonline:

- Kirigami support
- [Better ace editor integration with QML](https://github.com/ajaxorg/ace/pulls?q=author%3Apatrickelectric+)
- A qmlonline element to integrate with websites like Qt/KDE website documents

Any feature request will be much welcome!

# Thanks!

Special thanks to Arthur Turrini, everybody in [Qt Brasil telegram channel](https://t.me/qtbrasil) and the KDE members, the project would not be possible without the help and inspiration provided by the developers.
