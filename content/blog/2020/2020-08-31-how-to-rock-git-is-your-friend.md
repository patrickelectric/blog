+++
title = 'How to rock: Doing the right thing'
date = 2020-08-31T13:55:39z
draft = true
[taxonomies]
tags = [ "kde", "development", "tutorial" ]
[extra]
header = "/assets/how_to_rock/maintain.jpg"
+++

After working for some time collaborating with open source/free software projects, I started to help newcomers to contribute and push the development practices further with good practices, that can used for software collaboration and personal projects.


<!-- more -->

## General tips

I'll start throwing some good practices of code development and contribution, and during this post I'll explain why such practices can be archived without much effort:

### Development flow

- **Avoid creating multiple PRs**, update the ones that are still open.
  - E.g: You created a Pull Request called "Add button feature", some modifications will be necessary after the review process, and for that you'll need to update the same branch over creating new ones. That's necessary to help the project maintainers to see previous comments and your development history. Creating multiple PRs will only make the maintainers confuse and unable to track old comments, suggestions and your code changes between PRs.

- **Create atomic and self-contained commits**, avoid doing multiple things in the same commit.
  - E.g: You have created a commit to fix the serial communication class, and inside the same commit you are doing 3 different things, removing trailing spaces, fixing a pointer validation check and a typo in the documentation of a totally different class. This can appear to be silly and bureaucratic, but there are good reasons to break this simple commit and at least 3 different commits, one for the pointer check, a second one for the typo and a third one for the trailing space.
  - Developers usually track lines history to understand the changes behind a functionality, it's common to search with grep history from commits or lines changes in specific commits to understand the history of a library, function, class or a small feature, if the commits start to be polluted with unnecessary changes, this development practice will be almost impossible to be done, since a bunch of unrelated lines will me changed between commits and this technic will be unable to help the dear developer.
  - The example was also really simple, but you can imagine what happens if you change different parts of the code, for unrelated things, and a bug appears, technics such `git bisect` will still work, but the result will be much harder to understand which line is the one that created such bug.

- **Create atomic and self-sustained PRs**, avoid doing multiple things in the same PR, like different features. They may appear simple with small pieces of code/functionality but they can escalate quickly after a review process, and if both features are somehow related or dependently, it's recommended to break it in multiple PRs with code to maintain compatibility with current code base.
  - E.g: You have applied a PR for software notifications, and somehow you also added a URL fetch functionality to grab new software versions and etc. After the first review, the maintainer asks to create a more abstracted way to fetch api and to deal with network requirements, this will start to convolute the PR, moving the initial idea of the notification feature to an entire network REST API architecture. With that, it's better to break the PR in two, one that only provides the notification software and interface feature and a second PR that is used for the REST API related code.

- **Do your own review**, the final and probably most important tip of all, doing that will train your mind and eyes to detect poor code standards or bad practices, it'll also make your PR be merged easily and faster, since you'll be able to catch problems before the reviewer feedback. Some reviewers may thing that reviewing your own PR is a must, since we are talking about open source projects and free software, you should understand that the people that are reviewing your code are not obligated to do so, the majority are collaborating and donating their own time to help the development and maintenance of such projects, doing your own review is a sign of empathy about the project and maintainer time.

### Code practices

- **Encapsulate magic numbers**, it's much easier to explain that in code:
  ```cpp
  ...// First version
    start_communication(0, 1, 232);
    // Reviewer: What is the meaning of 0 ? What is 1 ? Why 232 ?

  ...// Second version
    enum class Messages {
        ...
        RequestStatus = 232,
    }
    ...
    uint8_t vehicle_id = 0;
    uint8_t component_id = 1;
    start_communication(vehicle_id, component_id, Messages::RequestStatus);
  ```
  As you can usee, the second versions makes everything more readable, we know that we are starting the communication with a vehicle that has an id of 0 and the vehicle probably contains a component that id is 1, and while calling this function we are also requesting the status. Much better than 0, 1 and 232 right ?
  This will help the reviewer to understand what is the meaning of such numbers and also developers that will touch this code in the future.

- **Avoid multiple arguments**, that's also an easier thing to explain with code:
    ```cpp
    ...// First version
    auto vehicle = new Vehicle(
        VehicleType::Car,
        4,
        {28, 31},
        Fuel::Electric,
        1.2,
        613
    );
    // Reviewer: What is the meaning of all this values ?
    //           How can we make it better ?

    ...// Second version
    auto vehicle = new Vehicle(VehicleType::Car)
    vehicle->setNumberOfTires(4);
    vehicle->setTirePressure({28, 31});
    vehicle->setFuel(Fuel::Electric);
    vehicle->setWeightInTons(1.2);
    vehicle->setAutonomyInKm(613);

    ...// It's also possible to use aggregate initialization in C++20
    auto vehicle = new Vehicle({
        .type = VehicleType::Car,
        .numberOfTires = 4,
        .tirePressure = {28, 31},
        .fuel = Fuel::Electric,
        .weightInTons = 1.2,
        .autonomyInKm = 613,
    });
    ```

    Both second and C++20 alternatives are valid for a better readability of the code, to choose between both alternatives will depend how are you going to design your API, probably if you are more familiar with the Qt library, the original second version appears to be the most common, the C++20 alternative appears to be a bit more verbose but can be useful to avoid multiple function calls and helpful when dealing with a simpler code base or not development an library.


<!--
```js
import QtQuick 2.7
import QtQuick.Controls 2.3

Rectangle {
    color: "red"
    anchors.fill: parent

    Text {
        text: "POTATO"
        font.pixelSize: 50
        color: "white"
        anchors.centerIn: parent
        RotationAnimator on rotation {
            running: true
            loops: Animation.Infinite
            from: 0
            to: 360
            duration: 1500
        }
    }
}
```

<iframe style="border-style: none;" height="200" id="frame" src="/assets/qmlonline_everywhere/frame.html" onload="frameLoaded();"></iframe>
<script>
  function frameLoaded() {
    const frame = document.getElementById("frame").contentWindow;
    frame.registerCall({
      posInit: function() {
        frame.setCode(
"import QtQuick 2.7; \
import QtQuick.Controls 2.3; \
Rectangle { \
    color: 'red'; \
    anchors.fill: parent; \
    Text { \
        text: 'Potato'; \
        font.pixelSize: 50; \
        color: 'white'; \
        anchors.centerIn: parent; \
        RotationAnimator on rotation { \
            running: true; \
            loops: Animation.Infinite; \
            from: 0; \
            to: 360; \
            duration: 1500; \
        } \
    } \
}"
        )
      },
    })
  }
</script>
-->

<!--
# How to have a beauty flow

<img src="/assets/how_to_rock/the_art_of_beauty.jpg" width="600">

<figcaption>"The Art of Beauty" by cogdogblog is licensed under CC BY 2.0</figcaption>

Git is awesome, git rocks, git is like a super advanced car, but if you don't read the manual, you'll never know the features!

The idea behind this section is to point some important things that I learned in the pass years while reading some great books, such as: [Mastering Git](https://isbnsearch.org/isbn/9781783553754) and [Pro Git(it's free!)](https://git-scm.com/book/en/v2).

## Tig

<div class="quote">"what's the use of having access to everything, if you can't visualize it."</div><br>

Git is really great, but what matters a great tool if the user interface is not as polished as we desire to be. **`Tig`** is one of the greatest tools to be used with Git, is the UI that mostly programmers are missing to visualize and understand what is going on in the git history. If you didn't know about it, install and use it now.

![image](/assets/how_to_rock/tig_all.png)

`Tig` can also allow a bunch of useful commands, such as `log`, `show`, `status`, `reflog`, `blame`, `grep`, `refs`, `stash` and others, we are going to talk about some of these later.

## Reflog

`git reflog` is one of the most important commands, when I saw its power for the first time it was like an epiphany, all commands and actions that I did in the entire history of my project was there, I could just checkout and see the history of my development tree in any moment, before or during a rebase, the history of a branch before a terrible idea in the code, everything is possible to recover or to start from a previous point, if have done any git command and the code is commited, you'll not loose it.

For an awesome experience, I recommend to use `tig` with `reflog` (`tig reflog`) to see a user-friendly history of `reflog`.

![image](/assets/how_to_rock/reflog.png)

<figcaption>checkout, rebase, reset, cherry-pick, the history is all there and you can checkout in any hash!</figcaption>

## Git add --patch / git add -p

![image](/assets/how_to_rock/add_p.png)
<figcaption style="margin-top: -2em">Add only what you really need</figcaption>

`git add -p` is a great command, as a good practice,
-->
