---
title: Building a Glowing Progress Indicator
subtitle: And learning about CustomPaint and Animations in the Process
date: 2023-09-02
tags: ["flutter", "ui", "mobile"]
---

Sometimes, small but carefully crafted UI elements elevate an application from good to outstanding and show the love for detail the developer has put into their app.

Today we are gonna look into one of those special widgets: a **Glowing Progress Indicator**

## Breaking up the Problem   
Tackling a complex UI component like an interactive, animated and glowing progress indicator all in one go quickly becomes overwhelming. Finding intermediate goals makes the whole process more approachable:   

1. Get an arc with variable degree of completeness 
2. Add the glow 🌟 effect
3. Add additional UI like text hints to it
4. Make it Interactivate 
5. And finally some sweet animations 🎬

## Building an arc
Whenever you need to draw custom shapes in flutter, you won't get around using the `CustomPaint` Widget. It allows you to draw circles, rectangles and also arcs, which are slices of a circle.    

To use `CustomPaint`, we first gotta create a class extending `CustomPainter`.
```dart
class _CirclePainter extends CustomPainter {
  _CirclePainter(this.progress);
  final double progress;

  @override
  void paint(Canvas canvas, Size size) {
	// currently not drawing anything
  }

  @override
  bool shouldRepaint(covariant _CirclePainter oldDelegate) {
    return progress != oldDelegate.progress;
  }
}
```

The `progress` parameter will be used to determine how much of the circle outline to draw. Let's get our hands dirty now and start painting:   

```dart
  void paint(Canvas canvas, Size size) {
    final fullCanvasRect = Rect.fromLTWH(0, 0, size.width, size.height);
    final strokePaint = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 20
      ..color = Colors.greenAccent;
	
    canvas.drawArc(
        fullCanvasRect, -.5 * pi, 2 * pi * progress, false, strokePaint);
  }
```

The `Paint()` object defines our brush - it's thickness, color, etc. - which we then pass as the last argument to `canvas.drawArc()`.
To understand the first equation `-.5 * pi` we must know, that 0° is normally not at the top but right side. Therefore we must rotate it backwards (=>`-`) a forth of a whole circle (which in radians, is `2*pi`).   
Hooray, the first step is done this is what our screen looks like now:
{{< figure src="./step1.jpg" class="centerimg">}}


## Where's the Glowup?
If we want to add a glowing ring to our app, we first gotta understand what **glow** really is. After playing some rider I had a pretty good guess.
1) Colorful, blurry background lights
2) a bright and sharp middle part   

Creating something blurry with `CustomPaint` seemed daunting at first, but with the help of `BackdropFilter` it is easily achieved.   
For the inner, sharp white part I copy-pasted our first `_CirclePainter`, gave it a new name and changed the *Brush* in `paint()`

```dart
final whiteSmall = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 5
      ..color = Colors.blue.shade100
      ..strokeCap = StrokeCap.round;
```   
Next up we got to `Stack` both on top of each other and add a blur-`BackdropFilter` in between to only affect the outer (now blue) stroke.
```dart
Stack(
        fit: StackFit.expand,
        children: [

          // The colorful part we want to get blurred a lot
          CustomPaint(
            painter: _OuterGlowPainter(progress),
          ),

          // Our big blur (higher sigmaX/Y means blurrier)
          BackdropFilter(
              filter: ImageFilter.blur(sigmaX: 5, sigmaY: 5),
              child: Container()),
		   
          // the practically white inner part
          CustomPaint(painter: _InnerLightTubePainter(progress)),
          
          // And a very light blur here 
          BackdropFilter(
              filter: ImageFilter.blur(sigmaX: 1, sigmaY: 1),
              child: Container()),
        ],
      ),
```
> **Remember:** The `BackdropFilter` only affects Widgets *higher up* the Stack.   

For a bit of contrast I also made the Apps background a dark gradient.
{{< details "> `Click here to see how`" >}}
To achieve this, we give the Container at the root a `BoxDecoration` with a diagonal `LinearGradient`.
```dart
class MainApp extends StatelessWidget {
  const MainApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Container(
          decoration: const BoxDecoration(
              gradient: LinearGradient(colors: [
            Color(0xff050a30),
            Color(0xff000c66),
            Color.fromARGB(255, 7, 7, 170),
          ], begin: Alignment.topRight)),
          child: const Center(
            child: GlowingProgressIndicator(progress: .5),
          ),
        ),
      ),
    );
  }
}
```
{{< /details >}}
This is where we are at now:
{{< figure src="./step2.jpg" class="centerimg">}}

## It's just a circle   
For our progress indicator to be more than just a circle we need text that helps the user understand, what the indicator's purpose is. Letting the developer pass down a child to the `GlowingProcessIndicator` class and then centering it makes this really easy and good looking.  

To further improve the UX and make it clear that the circle want's to be completed, not just an arc, I decided to add a small, black stripe around the entire circumference. The challenge was it not getting blurred, but also not laying on top.   
> **Remember:** The `BackdropFilter`'s child is never affected by the effect!   
```dart
BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 5, sigmaY: 5),
    child:  CustomPaint(
        painter: _FullCircleLeadPainter(),
      ),
    ),
```
Already looks a tad better!
{{< figure src="./step3.jpg" class="centerimg">}}

## Where's the Action? 💥
Currently our Widget isn't interactive - therefore also pretty useless. To fix this, I made the MainApp Stateful, added a `completedTodos` field that can be incremented with a `floatingActionButton`   

> **Attention:** This also means that the **entire** app gets re-rendered every time I increment the todo counter. That is fine for a small app like this, but for bigger projects try to keep your state at the lowest possible level. 
```dart
class _MainAppState extends State<MainApp> {
  final maxTodos = 12;
  int completedTodos = 1;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        floatingActionButton: FloatingActionButton(
          child: const Icon(Icons.check),
          onPressed: () => setState(() {
            completedTodos = (completedTodos + 1) % maxTodos;
          }),
        ),
        body: Container(
          decoration: ...,
          child: Center(
            child: GlowingProgressIndicator(
                progress: completedTodos / maxTodos,
				  child: ...),
```
{{< figure src="./step4.jpg" class="centerimg">}}

## The Flow is missing   
In order to polish the user experience even more, we should animate the state of the circle after completing a ToDo. I chose to go with the simplest option which was an `AnimationController` linearly interpolating between its lower- and upper bound. They respectively stand for the state before the click and the new progress.   
For the sake of re usability I separate the `AnimatedGlowingProgressIndicator` into an entirely new Widget.   
```dart
class _AnimatedGlowingProgressIndicatorState
    extends State<AnimatedGlowingProgressIndicator>
    with TickerProviderStateMixin {
  late AnimationController _animationController;

  void animateProgress(begin, end) {
    if (begin > end) {
      return;
    }
    _animationController = AnimationController(
        lowerBound: begin,
        upperBound: end,
        vsync: this,
        duration: const Duration(milliseconds: 400));
	 // We want the view to update whenever the controllers value changes
    _animationController.addListener(() {
      setState(() {});
    });
    _animationController.forward();
  }

  @override
  void initState() {
    // If the component gets re-loaded we also want it to animate back 
    // to the previous progress
    super.initState();
    animateProgress(0.0, widget.progress);
  }

  // This function is called whenever the widget gets a new progress
  @override
  void didUpdateWidget(covariant AnimatedGlowingProgressIndicator oldWidget) {
    
    super.didUpdateWidget(oldWidget);
    // Needed as I switch from 12 -> 0 and else we get an error 
    if (oldWidget.progress > widget.progress) {
      return;
    }
    // Also clean up ressources here
    _animationController.dispose();
    animateProgress(oldWidget.progress, widget.progress);
  }

  @override
  void dispose() {
    // Properly dispose the controller.
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
	// And then we return a normal Indicator with interpolated values
    return GlowingProgressIndicator(
        progress: _animationController.value, child: widget.child);
  }
}
```

## We made it
Looks like we have completed this project and arrived at a very satisfying end result. You can view the source code with all 5 steps as individual commits [here](https://github.com/the-lightstack/GlowingProgressIndicator). Thanks for following along and if I sparked your creativity, consider following [→ my newsletter🗞️ ←](https://buttondown.email/Lightstack?tag=article)😁
{{< figure src="./video.gif" title="Final result" class="centerimg">}}