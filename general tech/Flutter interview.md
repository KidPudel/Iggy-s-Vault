# Flutter

# Interview

Threads

in flutter no such thing as thread, there is “isolated” and is isolated doesn’t not share a memory, so therefore in flatter deadlock is not possible

[Top Flutter Interview Questions (2023) - InterviewBit](https://www.interviewbit.com/flutter-interview-questions/)

# top 10 interview questions

- **1. Зачем вообще использовать Flutter? Какие у него преимущества недостатки?**
    
    Own engine, dart, performance, native, rapid development ui building 
    
- **2. Чем отличается Stateful от Stateless виджет?**
    
    Statefull lets dynamic data show (update) present data, up to date, by rebuilding its part
    
    Stateless doen’t rebuilds everytime, it’s not listening to changes
    
- **3. Какие ещё существуют способы управления State?**
    
    At the fundament level there is inherited widget, which lets you sent data down the tree easily, by using provider, valuelistenable, changenotifier 
    
    BUT its just too much junk if app grows, soo…
    
    State management tools like BloC, which serves as a middle chain between UI and Data which describes the logic
    
- **4. Для чего применяются Keys?**
    
    To pass key owners context, state, parameters
    
    key is responsible for preserving the state of the widget
    
    primary use is to modify a widget tree that has stateful widgets
    
- **5. Какие типы Keys бывают?**
    
    UniqueKey
    
    GlobalKey
    
    ValueKey
    
- **6. Как Flutter работает «под капотом»?**
    
    App framework (dart),
    
     flutter (widget, render engine)
    
    platform (canvas, thread setup, event loop, render surface setup)
    
    - **Upper layers**: The Dart-based platform that takes care of app widgets, gestures, animations, illustrations, and materials;
    - **Flutter engine**: Handles the display and formatting of text.
    - **Built-in service**: Used for the management of plugins, packages, and event loops.
    
    Debug, jit just in time interpretet
    
    Release aot ahead of time compiled
    
- **What is BuildContext**
    
    BuildContext is basically what controls the position of a widget in a tree, locator of widgets, it tracks each widget in a tree and locates them **IN THE TREE**
    
- **7.  Компиляция — это, конечно, хорошо. Но что такое Widget, и как он отрисовывается?**
    
    widget is everything, **it’s a description of an element** (instance of a ui block), construct blocks , immutable blocks of UI, where as element manages the entity of a tree, so it declares and constructs, defines user interface
    
    Like a brick in a house Text(…
    
    And Widget Text, is definition and description of a brick as term
    
    If we need mutable entity we use State
    
    State not bounded to the Widget, but rather to element representation what we wrote (instance!
    
    Because one widget could be nested to the tree 10000 times, like text widget
    
    Element creates by widget.createElement()
    
    Element could stay unactive untile the end of a frame, after that its removed (unmounted)
    
    So element tree manages lifecycle and bounds
    
    Also there is a RenderObject, object of tree of visualization element that contains protocols of drawing and position of the widget
    
    There are 3 main trees of rendering is tree
    
    widget tree - resposable for decloration and storing properties of a widget
    
    Element tree - manages lifecycle and bounds widgets to hierarchy 
    
    Render objects - resposible for drawing and positioning and determine sizes and constraits
    
- **8. Поговорим о многопоточности. Что вы о ней знаете, и как она реализована на Flutter?**
    
    Dart is single threaded , but under the hood is 4 actually, but why for programmer 1?
    
    Because in a context of flutter every process, even an application is a thread. In dart it’s called Isolated, ***and difference between isolated and thread is that isolated does not share a memory and threads does not divide it, therefore for every isolated (main and additionals) there is Event Loop (therefore own MicroTask and Event queues) and its own memory space and communication is happening via messages on ports***
    
    when process is created dart does:
    
    1. Creates 2 queues MicroTask and Event
    2. executes main and by the end of the method execution launches EventLoop
    
    ***EventLoop has two 2 queues, first with big events and second for short tasks MicroTask***
    
    ***For a whole main process of the program EventLoop is controlling the order of the execution of our code depending on those 2 queues***
    
    its a description on how dart works in a single thread
    
- **9 async**
    
    asynchronous is working with marked **`async`** keyword, the execution of this method is sequential until `await` keyword, then it’s going to wait until `await` is done, and then completes normally,
    
    every async method marked with Future and it has then() and cathError()
    
- **10. А как же тогда запустить сложные вычисления отдельно от основного потока? Что такое Isolate?**
    
    Communication between isolated can be achieved via:
    
    1. low level solutions with Isolated and ports
    2. In case when we need to compute something on a different thread we can use `compute()` method, this will creates Isolated compute on callback passing necessary parameters and then returns the result of a callback and then kills the Isolated  
    **IMPORTANT:** 
        1. callback must be top level function (meaning outside of any class or anything, just directly in a file)
        2. Platform-Channel communication (between flutter and host platform) can only be done in main Isolate that is created by launching application, meaning it cannot be done via created by us Isolated
- **11. А работают ли принципы SOLID с Flutter?**
    
    Yup
    
    **Single responsibility** → class should has 1 reason to change and method 1 purpose. just clear and right organisation of the code
    
    **Open-Closed Principle** → must be opened for extension and closed for modifications. we can have abstract classes and we can inherit and also extension keyword, so we can use parents methods and add ours
    
    **Liskov Substitution** → subclass should be able to replace objects of a superclass without errors in a program
    
    **Interface Segregation** → interfaces should not force clients to implement something they don’t need, interfaces should not be monolithic (cleaner and more maintainable code)
    
    **Dependency Inversion** → high-level modules (application logic) should not depend on low-level modules (some specific implementations of some part like repositories) but both should depend on abstract (interface of abstract class) (lose coupling and dependency injection
    
- **KISS**
    
    keep it simple
    
- **DRY**
    
    dont repeat yourself
    
- **12. modes**
    
    debug - jit, logs
    
    profile - debugging level to analise performance (disabled in emulators)
    
    release - aot, disabled debugging
    
- **runApp vs main**
    
    main is a function where program starts, entry point
    
    runApp allows to return a widget, it is called in main and its a driver of an application
    
- **plugin vs package**
    
    plugin is written in native code
    
    package is written in dart
    
- **flutter inspector**
    
    tool that allows visualise blueprint of the widget
    
- **ticker**
    
    ticker tells how often animation is refreshed
    
- **mixin**
    
    dart doesnt have multiple inheritance, so use mixin, it allows us easily write reusable class code for multiple hierarchy levels
    
- **animations**
    
    # There is two types of animations
    
    1. Code-based animations that is ther to enhance/mutate the look and feel of for example base widgets like row or column like changing colors shapes and positions
    2. Drawn-based animations looks like someone has drawn them, it could be srite or transformation that is hart to do with code
    
    # Code-based animations
    
    They could be:
    
    - Implicit, rely on simnply setting a new value for some widget property
    - Explicit, explicit animations involve `AnimationController` and animated only when explicitly asked
    How to choose?
    - Does it repeat "forever"? (as long as it on screen or as long as condition is true)
    - Is it discontinous? (animation with a circle from small, to large, but not back, it just starts again, small large - small, large)
    - Do multiple widgets animated together?
    If any of those are yes, then your explicit widget
    
    ### Is there build-in widget?
    
    Imlicit, look for <Animated-"foo"<- where foo is ther property you are animating> like `AnimatedContainer`
    If there is no implicit build-in animation, use `TweenAnimationBuilder` to create an implicit custom animation
    
    Explicit, look for <"foo"-Transition>
    
    If there is no explicit build-in animation, ask, do i want stand alone customn animation, then use `AnimationWidget`, if you want it to be a part of existing surrounding widget usew `AnimationBuilder`
    
    # Implicit
    
    Implicit animations in Flutter are animations that are automatically handled by the framework when a specific property of a widget changes. When the property changes, the framework automatically animates the transition from the old value to the new value.
    
    For example, if you use a `AnimatedContainer` widget and change its `height`, `width`, `color`, or other properties using a `setState` or other state management mechanism, the framework automatically animates the transition between the old and new property values.
    
    ```dart
    AnimatedContainer(
      duration: Duration(seconds: 1),
      height: _isExpanded ? 200 : 100,
      width: _isExpanded ? 200 : 100,
      color: _isExpanded ? Colors.blue : Colors.red,
    ),
    
    ```
    
    # Explicit
    
    Explicit animations, on the other hand, require more manual control over the animation process. They are used when you want to animate a property that doesn't automatically trigger animations like implicit animations.
    
    To create an explicit animation, you typically use a `AnimationController` or `Tween` and use its value to manually update the animated property. You control when the animation starts, stops, and updates the values.
    
    ```dart
    class MyAnimatedWidget extends StatefulWidget {
      @override
      _MyAnimatedWidgetState createState() => _MyAnimatedWidgetState();
    }
    
    class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
        with SingleTickerProviderStateMixin {
      AnimationController _controller;
      Animation<double> _animation;
    
      @override
      void initState() {
        super.initState();
        _controller = AnimationController(
          vsync: this,
          duration: Duration(seconds: 1),
        );
        _animation = Tween<double>(begin: 0, end: 1).animate(_controller);
      }
    
      @override
      void dispose() {
        _controller.dispose();
        super.dispose();
      }
    
      void _startAnimation() {
        _controller.forward();
      }
    
      @override
      Widget build(BuildContext context) {
        return GestureDetector(
          onTap: _startAnimation,
          child: Opacity(
            opacity: _animation.value,
            child: Container(
              width: 100,
              height: 100,
              color: Colors.blue,
            ),
          ),
        );
      }
    }
    
    ```
    
    In this explicit animation example, we control the opacity of the `Container` using an `AnimationController` and a `Tween`. When the user taps on the widget, `_startAnimation` is called, which triggers the animation to run.
    
    **Choosing Between Implicit and Explicit Animations:**
    
    - Use implicit animations when you want a simple, straightforward animation that automatically handles property changes.
    - Use explicit animations when you need more control over the animation process, such as running animations based on user interactions or complex animation sequences.
    
    We could use implicit AnimatedContainer, to change the size of the button to extend it on a press, when adding to the basket.
    
    We could use explicit transition to play some animation while music is playing.
    
    Both implicit and explicit animations are powerful and can be used in combination to achieve complex and visually appealing animations in your Flutter app. It's essential to choose the right type of animation based on your specific use case and the level of control you need over the animation process.
    
    # Build-in simple implicit animations
    
    there are some widgets that are animated versions of existing widgets
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/c8427117-6238-487a-928f-823ee0f8c13a](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/c8427117-6238-487a-928f-823ee0f8c13a)
    
    Imagine you have a simple app with `Container` and a `Button`, and when you press the button, `Image` in container changes its size
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/52ff5802-686c-4418-b0d0-dca2df0627ee](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/52ff5802-686c-4418-b0d0-dca2df0627ee)
    
    But we can make it smooth by changing `Container` -> `AnimatedContainer`, adding duration, and as a result we get nice interpolation (the process of animating through the value between old and new)
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/1833153b-9bc5-47d5-8909-d2f763227c8f](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/1833153b-9bc5-47d5-8909-d2f763227c8f)
    
    Also you can change it's
    
    ```
    Curve
    ```
    
    , it changes with what rate values are changing, aslo you can add custom curves
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/4da74c74-d38b-4b3e-b223-e1786f3cdeb1](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/4da74c74-d38b-4b3e-b223-e1786f3cdeb1)
    
    Note that you don't have to put it in a `StatefullWidget`, you can put it in a stream or `FutureBuilder`
    
    # Custom implicit animations
    
    What if there is no `AnimatedFoo`??
    
    Don't worry, we can build it with `TweenAnimationBuilder`
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/acefa9c4-c5f1-4e76-bf21-c50eb76d1190](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/acefa9c4-c5f1-4e76-bf21-c50eb76d1190)
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/3f28835b-e01f-4b9f-9e2e-38a3ef5aaf29](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/3f28835b-e01f-4b9f-9e2e-38a3ef5aaf29)
    
    Also you can animate padding values, border, eleveation etc
    
    ## overview
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/9b9a82d7-00c3-4cbc-982a-ea5c61433326](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/9b9a82d7-00c3-4cbc-982a-ea5c61433326)
    
    # Build-in explicit animations
    
    Explicit animations gives you more controll.
    
    we could do something with AnimatedContainer and Transform, but it would turn and then stop...
    
    [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/2cc6bd14-2f66-4a2c-9f46-ce268aa37712](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/2cc6bd14-2f66-4a2c-9f46-ce268aa37712)
    
    With explicit widgets we have controll of a time, for that we need `AnimationController`
    
    `AnimationController` handles checking for ticks and gives us useful controlls of over what our animation doing.
    
    Since `AnimationController` also has its own state, we need to manage it with init and dispose
    
    ```dart
    @override
    void initState() {
       super.initState()
       _animationController = AnimationController(
       ...
       );
    }
    @override
    void dispose() {
       super.dispose()
       _animationController.dispose();
    }
    
    ```
    
    For `AnimationController` we must provide two parameters:
    
    1. `duration:` (how long is animation last) because we need an object that tells how far we are in a single rotation
    2. `vsync:` gives a reference to the object to notify about changes. The presence of vsync prevents offscreen animations from consuming unnecessary resources. You can use your stateful object as the `vsync` by adding `SingleTickerProviderStateMixin` to the class definition.
        
        [https://github.com/KidPudel/flutter-starter-kit/assets/63263301/fba4ff58-0142-451f-901f-2b53c3df238f](https://github.com/KidPudel/flutter-starter-kit/assets/63263301/fba4ff58-0142-451f-901f-2b53c3df238f)
        
- **null aware operator**
    
    ? ! ??
    
- **sizedbox vs container**
- how isolated communicate
    
    We can implement communication between two isolates using **ReceivePort and SendPort**
    
- new in flutter
    
    android engine got updated 
    
- new in dart
    
    sealed class
    
- responsive interface
    
    adjust by dimensions
    
- adaptive interface
    
    layout specifically for the platform
    

[10 популярных вопросов, которые нужно знать, чтобы пройти собеседование на позицию Flutter-разработчика / Habr](https://habr.com/ru/amp/publications/708692/)

# Helpful info

## setState

[What is setState() in flutter and when to use it?](https://dev.to/nicks101/when-to-use-setstate-in-flutter-380#setsate-use)

## Going one step ahead :

- What happens when you use the `setState` inside the `onSaved` function only?

```dart
...
onSaved: (value) {
      /// updating userText variable
      if (value != null) userText = value;
      setState(() {});
    },
  ),
),
OutlinedButton(
  onPressed: () {
// validating form
    if (!_formKey.currentState!.validate()) {
      return;
    }

// saving form
    _formKey.currentState!.save();

// updating hasSubmitted
    hasSubmitted = true;
  },
...

```

It works here also. Surprise!!

But why? This goes against everything we've read so far in this blog.

So here's what happens.

When we call `setState`, the Widget inside we called it is marked as `dirty`.

Now whenever the framework actually rebuilds the UI of the screen, it will take into account all the latest values of the respective variables and paint the pixels on the screen.

> This happens 60 times per second usually, which is the frame per second (fps) rate of flutter. That means approximately every 16ms (1000/60 ms).
> 

If there is any other change until the next frame renders, those changes will also be reflected on the screen's UI.

The change in `hasSubmitted` variable falls under such case.

How do we verify it?

Let's add print statements and see exactly when does the UI actually rebuild.

```dart
import 'package:flutter/material.dart';

class MyForm extends StatefulWidget {
  const MyForm({Key? key}) : super(key: key);

  @override
  _MyFormState createState() => _MyFormState();
}

class _MyFormState extends State<MyForm> {
  String userText = "";

  bool hasSubmitted = false;

  // for getting access to form
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    print('Widget build called');

    return Column(
      children: [
        Form(
          // attaching key to form
          key: _formKey,
          child: TextFormField(
            onSaved: (value) {
              print('inside save');

              if (value != null) userText = value;

              print('hasSubmitted value before setState: $hasSubmitted');

              setState(() {});

              print('hasSubmitted value after setState: $hasSubmitted');
            },
          ),
        ),
        OutlinedButton(
          onPressed: () async {
            print('button clicked: ->');

            // validating form
            if (!_formKey.currentState!.validate()) {
              return;
            }

            print('before calling save');

            // saving form
            _formKey.currentState!.save();

            print('after calling save');

            print('hasSubmitted value after calling save: $hasSubmitted');

            // updating hasSubmitted
            hasSubmitted = true;

            print(
                'hasSubmitted value after updating hasSubmitted: $hasSubmitted');
          },
          child: const Text('Submit'),
        ),
        if (hasSubmitted) Text(userText)
      ],
    );
  }
}

```

With these print statements, we can see the order in which the flutter framework updates the UI.

Let's write something in the text field and click on submit button.

Inside console:

```dart
Widget build called
button clicked: ->
before calling save
inside save
hasSubmitted value before setState: false
hasSubmitted value after setState: false
after calling save
hasSubmitted value after calling save: false
hasSubmitted value after updating hasSubmitted: true
Widget build called

```

Clearly, the Widget is built after the `hasSubmitted` is set to `true`.

### BUT setState must be called after we altered what we need so all what we need is marked as dirty and would be refreshed in UI, so future can drag some changes behind, so element would not refresh

So the major question is: "Why can't we use `setState` inside `onSaved` function instead of inside `onPressed` function, it seems to work fine."

Because this method doesn't work for all the use cases. And it is also logically wrong.

When you'll come back to refactor the code (like for adding a new feature), things might not work as expected. Debugging the code will also be difficult since the problem is in the old code.

Let's see an example of such a use case.

> Okay, we're almost done. This is the most important part. We've been building up to this point since the starting of this blog.
> 

Let's go back to the steps of this example and add one more thing to it.

- After saving the form, say we want to make an API call to send the data that the user typed to a server. For simplicity, let's say we're going to do some computation with the `userText` data which will take a minimum of 20ms (this can be any duration of your choice).

This is a practical example. Usually, we do communicate with our application's backend server.

Does our desired result still happen?

```dart
onPressed: () async {
  print('button clicked: ->');

  // validating form
  if (!_formKey.currentState!.validate()) {
    return;
  }

  print('before calling save');

  // saving form
  _formKey.currentState!.save();

  print('after calling save');

  print('hasSubmitted value after calling save: $hasSubmitted');

  /// some computation that takes 20ms
  await Future.delayed(const Duration(milliseconds: 20), () {});

  // updating hasSubmitted after 20ms
  hasSubmitted = true;

  print(
      'hasSubmitted value after updating hasSubmitted: $hasSubmitted');
},

```

See inside `onPressed` function. We're `awating` a `Future` and then updating the `hasSubmitted` value.

> If you don't know what await, async and Future means, then just think that we're basically saying to program, "Hey flutter framework, we're gonna do some task that'll probably take some time. Please do it in the next iteration."
> 
> 
> I'll encourage you to read about [asynchronous programming](https://dart.dev/codelabs/async-await). This concept is not exclusive to dart.
> 

### Note: We can also use a [Timer](https://api.dart.dev/stable/2.12.0/dart-async/Timer-class.html) to get the same effect, but here we'll use [Future](https://api.dart.dev/stable/2.13.3/dart-async/Future-class.html).

Inside console:

```dart
Widget build called
button clicked: ->
before calling save
inside save
hasSubmitted value before setState: false
hasSubmitted value after setState: false
after calling save
hasSubmitted value after calling save: false
Widget build called
hasSubmitted value after updating hasSubmitted: true

```

Now, we don't see the UI updates on the screen. And according to our print statements order, `hasSubmitted` variable is updated after the Widget has been rebuilt.