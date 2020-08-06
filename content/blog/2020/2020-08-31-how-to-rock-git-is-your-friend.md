+++
title = 'How to rock: Doing the right thing'
date = 2020-08-31T13:55:39z
draft = true
[taxonomies]
tags = [ "kde", "development", "tutorial", "how-to-rock" ]
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

- **Encapsulate magic variables**, it's much easier to explain that in code:
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
    uint8_t vid = 0;
    uint8_t cid = 1;
    start_communication(vid, cid, Messages::RequestStatus);
    // Reviewer: What is vid ? What is cid ?

  ...//Final version
    ...
    uint8_t vehicle_id = 0;
    uint8_t component_id = 1;
    start_communication(vehicle_id, component_id, Messages::RequestStatus);
  ```
  As you can usee, the second versions makes everything more readable, we know that we are starting the communication with a vehicle that has an id of 0 and the vehicle probably contains a component that id is 1, and while calling this function we are also requesting the status. Much better than 0, 1 and 232 right ?
  This will help the reviewer to understand what is the meaning of such numbers and also developers that will touch this code in the future.
  As you say in the last review, you also should avoid variables that are single variables or really short abbreviations, is much harder to understand this: $$C^2 = A^2 + B^2$$
  over this: $$hypotenuse^2 = catheti_1^2 + catheti_2^2$$

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

- **Encapsulate code when necessary**, try to break functions in a more readable and explanatory way.
    ```cpp
    ...// Original version
    bool Serial::start_serial_communication() {
        // Check if we are open to talk
        if (!_port || _port->register() != 0xb0001) {
            log("Serial port is not open!");
            return false;
        }

        // Send a 10ms serial break signal
        _port->set_break_enabled(true);
        msleep(10);
        _port->set_break_enabled(false);
        usleep(10);
        // Send intercalated binary for detection
        _port->write('U');
        _port->flush();

        // Send start AT command
        _port->write("AT+start");
    }

    // Reviewer: Try to make it more readable encapsulating
    //            some functionalities

    ...// Second version
    bool Serial::start_serial_communication() {
        if (!is_port_open()) {
            log("Serial port is not open!");
            return false;
        }

        force_baudrate_detection();
        send_message(MESSAGE::Start)
    }
    ```
    As you can see, the reason behind each block of code is much more clear now, and with that, the comments are also not necessary anymore, the code is friendly and readable enough that's possible to understand it without any comments.

- **Avoid comments**, that's a clickbait, comments are really necessary, but they may be unnecessary when you are doing something that's really obvious, and sometimes when something is not obvious, it's better to encapsulate it.
    ```cpp
        ...// Original version
        void blink_routine() {
            // Use the LED builtin
            const int led_builtin = LED_BUILTIN;
            // Configure ping to output
            setPinAsOutput(led_builtin);

            // Loop forever
            while(true) {
                // Turn the LED on
                turnPinOn(led_builtin);
                // Wait for a second
                wait_seconds(1);
                // Turn the LED off
                turnPinOff(led_builtin);
                // Wait for a second
                wait_seconds(1);
            }
        }
    ```
    Before checking the final version, let me talk about some points:
      - For each line of code that you see in this example you'll have a comment (like a parrot that repeat what we say), and the worst thing about these comments is that the content is exactly what you can read from the code! You can think that this kind of comment is dead code, something that has the same meaning as the code, but it does not run, resulting in a duplicated amount of lines, to maintain, if you forget to update each comment for each line of code, you'll have a comment that does not match with the code, in this will be pretty confuse for someone that's reading this code.
      - One of the most important skills about writing comments, is to know when not to write it!
      - A comment should bring a value for the code, if you can remove it and the code is understandable by a novice, the comment is not important.
      - The could need to express what is the desired behaviour, to make it easy to understand and maintain it.

    ```cpp
    ...// Final version
        void blink_routine() {
            const int led_builtin = LED_BUILTIN;
            setPinAsOutput(led_builtin);

            while(true) {
                turnPinOn(led_builtin);
                wait_seconds(1);
                turnPinOff(led_builtin);
                wait_seconds(1);
            }
        }
    ```

    There is a good video about this subject by [Walter E. Brown in cppcon 2017, "Whitespace ≤ Comments ＜＜ Code"](https://www.youtube.com/watch?v=NLebZ3XT92E).

    And to finish, you should not avoid comments, you should understand when comments are necessary, like this:
    ```cpp
    // From ArduPilot - GPIO_RPI
    void set_gpio_mode_alt(int pin, int alternative) {
        // **Moved content from cpp for this example**
        const uint8_t pins_per_register = 10;
        // Calculates the position of the 3 bit mask in the 32 bits register
        const uint8_t tree_bits_position_in_register = (pin%pins_per_register)*3;
        /** Creates a mask to enable the alternative function based in the following logic:
        *
        * | Alternative Function | 3 bits value |
        * |:--------------------:|:------------:|
        * |      Function 0      |     0b100    |
        * |      Function 1      |     0b101    |
        * |      Function 2      |     0b110    |
        * |      Function 3      |     0b111    |
        * |      Function 4      |     0b011    |
        * |      Function 5      |     0b010    |
        */
        const uint8_t alternative_value =
            (alternative < 4 ? (alternative + 4) : (alternative == 4 ? 3 : 2));
        // 0b00'000'000'000'000'000'000'ALT'000'000'000 enables alternative for the 4th pin
        const uint32_t mask_with_alt = static_cast<uint32_t>(alternative_value) << tree_bits_position_in_register;
        const uint32_t mask = 0b111 << tree_bits_position_in_register;
        // Clear all bits in our position and apply our mask with alt values
        uint32_t register_value = _gpio[pin / pins_per_register];
        register_value &= ~mask;
        _gpio[pin / pins_per_register] = register_value | mask_with_alt;
    }
    ```
    Mostly of the lines in this code can be impossible to understand without access or reading the datasheet directly,
    the comments help the programmers to understand what is going on and why, without the necessity of reverse engineer of the code.

- **Final and generic tip**:
  1. Try not to hinder other developers, make sure that:
      1. Code compiles/runs before sending a PR (Pull-Request).
      2. Code passes all tests (if they exist) before committing.
      3. If you need to check in broken or incomplete code, use a branch, or somehow minimize the impact on other developers.

  2. Commit code that is neat, portable, and documented:
      1. Use appropriate, descriptive names for classes and variables.
      2. Use doxygen or equivalent comments for every class and method, explain the purpose and intent of the class and how it fits into the overall architecture when writing docs.
      3. Remove extraneous code that is not used (classes, and methods in classes).
      4. It's also recommended to write short, to-the-point methods that encapsulate a very specific behavior, rather than long procedural functions.
      5. If your methods are longer that 30-40 lines of code, or if they have extensive conditional blocks, switch statements or indentation levels, they might be broken up into several methods. But this is very subjective. Related to this is the importance of factoring out common procedures into their own classes or methods.
      6. If you find yourself writing the same type of functionality multiple times, it's definitely time to refactor.

  3. As a software developer, you always should have in mind that software should be:
      1. Do not sacrifice maintainability over performance when not necessary
      2. Less readable code result in more bugs and wrong usages
      3. Make the code easy to newcomers
      4. You're not the only one working in your code, if you think that you are, say that to the future version of yourself.

## References
- [CppCon 2019: Kate Gregory “Naming is Hard: Let's Do Better”](https://www.youtube.com/watch?v=MBRoCdtZOYg)
- [CppCon 2018: Kate Gregory “Simplicity: Not Just For Beginners”](https://www.youtube.com/watch?v=n0Ak6xtVXno)
- [CppCon 2018: Kate Gregory “What Do We Mean When We Say Nothing At All?”](https://www.youtube.com/watch?v=kYVxGyido9g)
- [CppCon 2017: Lars Knoll “Qt as a C++ Framework: History, Present State and Future”](https://www.youtube.com/watch?v=YWiAUUblD34)
- [CppCon 2017: Kate Gregory “10 Core Guidelines You Need to Start Using Now”](https://www.youtube.com/watch?v=XkDEzfpdcSg)
- [API Design Principles - TQtC](https://wiki.qt.io/API_Design_Principles)
- [Designing Qt-Style C++ APIs - Matthias Ettrich](https://doc.qt.io/archives/qq/qq13-apis.html)
- [The Little Manual of API Design - Jasmin Blanchette](https://people.mpi-inf.mpg.de/~jblanche/api-design.pdf)


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
