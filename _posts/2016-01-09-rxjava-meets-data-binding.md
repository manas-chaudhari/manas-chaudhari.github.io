---
layout: post
title: "RxJava meets Data Binding (Part 1)"
date: "2016-01-09 20:23"
excerpt: >
  Getting values from RxJava's Observables to Views is a problem and often requires boilerplate.
  We explore how RxJava and Data Binding can be combined to solve this problem. A crude implementation of the `RxObservableField` class is presented.
---

> This is Part 1 about combining RxJava and Data Binding. This article presents the first iteration and how this pattern came into existence. [Part 2]({% post_url 2016-08-09-rxjava-meets-data-binding-part-2 %}) provides an improved internal implementation and approach for handling several practical issues.

RxJava and Android Data Binding both provide mechanisms for subscribing for changes. In many discussions, I find people recommending one against the other. However, I found that using Data Binding and RxJava works quite well.

## Before Data Binding
Before data binding, a convenient way for writing View Models was

```java
class CartModel {
  final Observable<Float> totalAmount; // Lets say this gets updated through a source that we don't care about
}

class CartViewModel {
  final Observable<String> totalAmountText;

  CartViewModel(CartModel cartModel) {
    totalPrice = cartModel.totalAmount.map { amount -> "$ " + amount.toString() };
  }
}
```

The view (which could be an Activity/Fragment or a custom widget, but that's not the point), would look like:

```java
class CartView {
  TextView totalAmountTextView;

  List<Subscription> subscriptions = new ArrayList<>();

  void init(CartViewModel cartVM) {
    subscriptions.add(cartVM.totalAmountText.subscribe { text -> totalAmountTextView.setText(text) });
  }

  void onDestroy() {
    for (subscription : subscriptions) {
      subscription.unsubscribe();
    }
  }
}
```

## With Data binding
Databinding allows you to bind object fields to almost _any_ property of a view through XML, which means no need to call setter methods like `setText()`. Instead of calling these methods, we can bind the `text` property to a field of view model.
This can be achieved as follows:

```xml
cart.xml

...
  <data>
    <variable name="vm" type="com.example.CartViewModel"/>
  </data>
  <TextView
    android:text="{ vm.totalAmountText }"/>
...

// In code which inflates cart, we do:
cartBinding.setVm(cartViewModel);
```

This is all fine except that databinding expects that `CartViewModel` must satisfy one of these conditions:
1. have a field `totalAmountText` of type `String` OR
2. have a method `String getTotalAmountText()` OR
3. have a field `totalAmountText` of type `ObservableField<String>` OR
4. _there are other ways, which aren't required in this context._

In the first two options, in order to make `totalAmountText` changes appear on the view, we'll need to make `CartViewModel` extend `databinding.BaseObservable` and invoke `notifyPropertyChanged(BR.totalAmountText)` whenever it changes. However, this adds boilerplate in the View Model code as we'll transition from

```java
class CartViewModel {
  final Observable<String> totalAmountText;

  CartViewModel(CartModel cartModel) {
    totalPrice = cartModel.totalAmount.map { amount -> "$ " + amount.toString() };
  }
}
```
to

```java
class CartViewModel {

  CartViewModel(CartModel cartModel) {
    cartModel.totalAmount.subscribe { amount -> setTotalAmountText("$ " + amount) };
  }

  String getTotalAmountText() {
    return mTotalAmountText;
  }

  String setTotalAmountText(String totalAmountText) {
    mTotalAmountText = totalAmountText;
    notifyPropertyChanged(BR.totalAmountText);
  }
}
```

Boilerplate is obviously undesirable. This brings us to option 3 i.e. `ObservableField`. `ObservableField` and `rx.Observable` are very similar in the sense that they allow subscribing for change. We can extend `ObservableField` to add a constructor which takes `Observable` as an argument.

```java
class RxObservableField<T> extends ObservableField<T> {
  public RxObservableField(Observable<T> source) {
    source.subscribe { value -> set(value) };
  }
}
```

With `RxObservableField`, our view model becomes very neat.

```java
class CartViewModel {
  final RxObservableField<String> totalAmountText;

  CartViewModel(CartModel cartModel) {
    totalPrice = new RxObservableField(cartModel.totalAmount.map { amount -> "$ " + amount.toString() });
  }
}
```

This implementation of `RxObservableField` is very basic. Few things need to be taken care of:
1. Memory Leaks
2. Closing subscriptions

## What's next?
1. As `RxObservableField` internally subscribes to an observable, this subscription should be closed when the view gets destroyed.
2. When `subscribe` method is called, the reference of `RxObservableField` instance goes to the source `Observable`. This might cause a leak.
3. `ObservableField` also allows consumers to set inner value. Thus, its more similar to a `Subject`. This would be useful to capture user input, for example in `EditText`. However, as data binding doesn't exactly support two way binding, some more work is required on that front.
4. Another thing is events like `onClick` or `onTextChanged`. It would be awesome to derive an `rx.Observable` for these events. However, writing a generic adapter seems tricky at this point.
