# Flutter

Developers having a hard time to choose Flutter architecture patterns. So Let's clarify some points before working with one.
There are different architecture patterns in a flutter:

- Vanilla
- Bloc
- ScopedModel
- Redux

#### Basic Difference:
Scoped Model is a way of passing Models down the Widget tree to descendants that inherit it by either using ScopedModelDescendant or by using the more traditional approach of ScopedModel.of. A Model is just a ChangeNotifier which is an object that notifies its listeners when something changes, which in this case are the values in your model.
BLoC pattern was created by the Google AdWords team to reuse code between their Flutter team and the angular dart team. It essentially works by exposing streams of values in the BLoCs which can be subscribed to by StreamBuilders which will rebuild every time a new value is added to the Stream. It has the benefit of being composable which works well with Flutter.
The Redux package brings the functionality of the Redux architecture to Flutter. I haven't used it so I can't say much about it, and there are a ton of resources online. There are also a couple of open source apps written with it so you can take a look at those too. Like this one which has been published in the play store: inkino

### Vanilla Pattern
In this pattern business logic and UI, layer decisions are placed in one file and the whole widget tree is gonna build every time a new update comes by calling setState(). 

> setState notifies the framework that the internal state of this object has changed and due to that framework will schedule build event in event-loop for this state.
> what setState does is it sets the flag to dirty and due to it in the rebuild method will get called.
> Also do not try to block UI thread by doing intense work in the setState method. for hack you can also change object outside ```setState()``` and call ```setState(){}``` empty it does the same thing except that your not blocking any thread. Sometimes I read that deserialize JSON object to model takes UI to miss the 16ms time frame and you need to use async or RxDart but it's not true. When using the async function we are not going to stuck technically unless it's heavy lifting like disk write operation or something. 
> Some times you might lead to a state where a screen has been disposed/(never inserted) but due to async callback you will get the error on a log that ```setState``` called after dispose/(widget that hasn't been inserted into widget tree). so check ```mounted``` flag before calling for an update.

- Pros

1. Easy to learn and understand.
2. No third-party libraries are required.

- Cons

1. The whole widget tree is rebuilt every time widget state changes.
2. It’s breaking the single responsibility principle. Widget is not only responsible for building the UI, but it’s also responsible for data loading, business logic, and state management.
3. Decisions about how the current state should be represented are made in the UI declaration code. If we would have a bit more complex state code readability would decrease.

### Bloc pattern
[Flutter patterns and bloc pattern](https://medium.com/flutterpub/architecting-your-flutter-project-bd04e144a8f1)




#### ScopedModel
#### Redux

###### Reference Links
1. [Provider pattern over bloc pattern]
2. [summarize the difference between popular Flutter architectures]
3. [Some architecture samples]
4. [Some more samples for architecture]
5. [Architecture comparision]
6. [Semantics in flutter]
7. [Flutter internals]
8. [Useful site for flutter articles]

[Provider pattern over bloc pattern]:<https://www.reddit.com/r/FlutterDev/comments/bmrvey/so_is_provider_recommended_over_bloc_just_watched/>
[summarize the difference between popular Flutter architectures]:<https://www.reddit.com/r/FlutterDev/comments/9qwvjj/can_anyone_summarize_the_difference_between/>
[Some architecture samples]:<https://fluttersamples.com>
[Some more samples for architecture]:<https://github.com/brianegan/flutter_architecture_samples>
[Architecture comparision]:<https://www.didierboelens.com/2019/04/bloc---scopedmodel---redux---comparison/>
[Semantics in flutter]:<https://www.didierboelens.com/2018/07/semantics/>
[Flutter internals]:<https://www.didierboelens.com/2019/09/flutter-internals/>
[Useful site for flutter articles]:<https://www.didierboelens.com>
---