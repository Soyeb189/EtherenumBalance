
# Adapting your app for foldable phones.

Mobile device concept is one step ahead of the trend in technology, bringing an enhanced visual experience to the palm of our hands with Foldable phones. This new form factor allows users to “stretch” the screen to a double size of a standard phone for better visualization, and also “reduce” the screen size to fit into your pocket as well.

Most of the well-known manufacturers have released several foldable device models tailored to different preferences. Hence, mobile apps would offer a user experience that might fit more than one fixed size of screen for a single device. Furthermore, they could take advantage of this to offer an immersive multimedia experience.

Consequently, apps must avoid wasting screen size with empty space that could be occupied by an informational or interactive element, namely button, text, panel, menu, etc.


# Programming tools

To make apps foldable-aware, Google has provided some tools with Android Jetpack within a new WindowManager API that enables apps to benefit from screen size change. The official [documentation](https://developer.android.com/guide/topics/large-screens/learn-about-foldables) is good enough to introduce the concepts, however, it isn’t enough for a hands-on adoption.

Though there are some thorough articles and sample projects, they are too complex to understand the basics of the API and its mechanisms. Like all hardware and sensor events, we can use several techniques to detect screen changes asynchronously; so Android Jetpack provides a high level API to hook with user events through three different asynchronous programming approaches: Callbacks, Kotlin Flow and RxJava. Most of the sample code we can find on the web uses either the old school callbacks, or Flow — which is not considered an API ready for production — and also, there are many Android apps that haven’t implemented Kotlin-first philosophy. Thus, this article aims to give a concrete, yet simple guide to implement foldable-aware features using RxJava.



## Code Section

Indeed, a foldable phone screen might have two states, open (flat) and closed (folded). But most foldable devices bring the capability of another quite useful intermediate state when the device is partially folded: Half-opened, also called Flex mode. This state is of a special interest because it allows users to have two logical screens with a single big screen, so that some UI elements such as controls or menu can be placed on one of them and the content itself is shown on the other one for instance. Therefore our apps should listen for these events and make changes on the UI to adapt or optimize the interaction.

First of all we should add the dependency in app module build.gradle file

```bash
  androidx.window:window-rxjava2:$windowVersion
```

Then it is recommended to make activities resizeable by declaring the setting in manifest:

```bash
  android:resizeableActivity
```

Now we are able to use the API to listen for fold states. We need an instance of WindowRepository that can be obtained from context

```bash
  context.windowInfoRepository()
```

Finally we can use that instance to observe the stream of state changes

```bash
  disposable = windowInfoRepository
                .windowLayoutInfoObservable()
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(callback)
```

As we can see, we should make sure we can dispose the stream as soon as it is necessary by calling disposable.dispose() as a good practice when working with RxJava. Likewise, since the observer will be called by a background thread and we need to update the UI it is necessary to switch to the main thread. The callback receives as a parameter an instance of WindowLayoutInfo which contains all the information needed inside the displayFeatures field, though its elements should be casted to FoldingFeature. The state field can be FoldingFeature.State.HALF_OPENED or FoldingFeature.State.FLAT. When displayFeatures is empty the screen is folded (closed); we should consider that state as a standard device screen.

So let’s say your app has a RecyclerView with large items that have been listed as a LinearLayout. But now when the user unfolds the device you can have a grid of elements. We could change its layout manager according to the state changes.

So the full code that implements this logic is the following:

```bash
  windowInfoRepository
   .windowLayoutInfoObservable()
   .observeOn(AndroidSchedulers.mainThread())
   .subscribe{ windowLayoutInfo ->
     windowLayoutInfo.displayFeatures.filterIsInstance(FoldingFeature::class.java)
       .firstOrNull ()?.let { foldingFeature ->
           if (foldingFeature.state == FoldingFeature.State.HALF_OPENED) {
               recyclerView.layoutManager = LinearLayoutManager(
                   context,
                   LinearLayoutManager.HORIZONTAL, 
                   false
               )
           } else {
               recyclerView.layoutManager = GridLayoutManager(
                   context,
                   3,
                   LinearLayoutManager.VERTICAL,
                   false
               )
           }
       }
}
```



This way we get a simple and basic foldable app that helps to understand and implement the functionality with an easy approach. But for sure, there should be some more logic around it considering hinge, device orientation while the screen is fully open, in vertical (Book mode) and landscape (TableTop mode). Also, we can optimize more precisely by calculating WindowMetrics as well as changing constraint layout settings or by using the ReactiveGuide component with animated transitions.

All of these depend on the user experience your app should offer and the content it is meant to show, whether video, text or controls. There are several UX design guidelines for foldables you should consider for your app.

One final note is that I’d recommend using feature toggles and A/B testing when developing first foldable-aware features. In Particular, the app should send the device model to the backend that it is consuming services from, so that it can deliver exclusive content for foldable devices, for example videos with higher resolution.




