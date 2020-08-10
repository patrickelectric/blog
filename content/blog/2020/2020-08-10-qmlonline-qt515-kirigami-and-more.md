+++
title = 'QML Online - Qt 5.15, Kirigami and more!'
date = 2020-08-10T13:55:39z
[taxonomies]
tags = [ "kde", "qml", "qt", "webasm", "web", "internet", "online" ]
[extra]
header = "/assets/qmlonline_kirigami_qt515/banner.jpg"
+++

I'm happy to announce that [QML Online ](https://qmlonline.kde.org) is now running with the last version of Qt (5.15) and with a initial Kirigami integration!

<!-- more -->

![image](/assets/qmlonline_kirigami_qt515/banner.jpg)

There is also a couple of updates for quality of life, like:
- HTML fixes/corrections
- Better integration with Firefox
- New Qt version information label
- Support for `QtQuick.XmlListModel`

But sadly, with new features we do have new bugs! As I said before, the Kirigami integration is a initial version, there are some know bugs with it, like:
- Breeze icons are not integrated
- OverlayDrawer has a transparent background (the issue appears to be common in low performance environments)
- Kirigami version is a bit old (v5.70), newer versions need Qt future feature for QFuture and friends

# What is next

I'll be working closer with Kirigami to fix these bugs, and the new feature of multiple instances of QML Online on the same webpage will be in hold for now.

As a reminder, please be free to send [Merge Requests](https://invent.kde.org/webapps/qmlonline/-/merge_requests), [feature requests, opinions and issues](https://invent.kde.org/webapps/qmlonline/-/issues/new).

# Thanks

I would like to thank all users of QML Online, and the people that are sending kind works about how it's improving their workflow and how useful the tool is! That really helps to move the project forward.
