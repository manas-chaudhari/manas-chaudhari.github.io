---
layout: "post"
title: "RxJava meets Data Binding (Part 2)"
date: "2016-08-09 20:56"
---

[Part 1]({% post_url 2016-01-09-rxjava-meets-data-binding %}) showed the origin of `RxObservableField` and its benefits. This part improves the implementation to solve critical issues like cleaning up of subscriptions/memory leaks.

```java
class RxObservableField<T> extends ObservableField<T> {
  public RxObservableField(Observable<T> source) {
    source.subscribe { value -> set(value) };
  }
}
```

One approach is to store this subscription, add a `close()` method to unsubscribe and invoke it for all fields inside ViewModel. However, that will add more boilerplate code.

## Different approach

Instead of subscribing at initialization, it would be great to subscribe only when the field is being observed and unsubscribe when it isn't. This mechanism auto cleans the subscriptions and is also more efficient as subscription is created only when required.

> This approach feels analogous to how we code in Functional Programming/Rx. Elements define what to do in terms of functions, different functions are composed to form a pipeline, and then it is activated only when the caller invokes. When inactive, the code is not doing anything.

```java

public class RxObservableField<T> extends ObservableField<T> {
  final Observable<T> source;
  final HashMap<OnPropertyChangedCallback, Subscription> subscriptions = new HashMap<>();

  protected RxObservableField(@NonNull Observable<T> source) {
    super();
    this.source = source
    .doOnNext(new Action1<T>() {
      @Override
      public void call(T t) {
        set(t);
      }
    })
    .share();
  }

  @Override
  public synchronized void addOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
    super.addOnPropertyChangedCallback(callback);
    subscriptions.put(callback, source.subscribe());
  }

  @Override
  public synchronized void removeOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
    super.removeOnPropertyChangedCallback(callback);
    Subscription subscription = subscriptions.remove(callback);
    if (subscription != null && !subscription.isUnsubscribed()) {
      subscription.unsubscribe();
    }
  }
}
```

Invoking `share()` on source `Observable` ensures that there is only a single subscription on it. Also, `share()` takes care of subscribing to the source on first subscription and unsubscribing when there are no subscribers.


## Problems in Two Way Binding

When capturing user input, Data Binding invokes the `set()` method to update the value inside ObservableField. Problem is that the value of the field will become inconsistent with the value from `source`. One way to solve this is to not allow `Observable` source for Two Way Binding fields. That way, there won't be an issue of inconsistency. Plain `ObservableField` can be used and we can convert an `ObservableField` to an `Observable` through a utility method.

```java
public class FieldUtils {
  @NonNull
  public static <T> Observable<T> toObservable(@NonNull final ObservableField<T> field) {
    return Observable.create(new Observable.OnSubscribe<T>() {
      @Override
      public void call(final Subscriber<? super T> subscriber) {
        subscriber.onNext(field.get());
        final OnPropertyChangedCallback callback = new OnPropertyChangedCallback() {
          @Override
          public void onPropertyChanged(android.databinding.Observable observable, int i) {
            subscriber.onNext(field.get());
          }
        };
        field.addOnPropertyChangedCallback(callback);
        subscriber.add(Subscriptions.create(new Action0() {
          @Override
          public void call() {
            field.removeOnPropertyChangedCallback(callback);
          }
        }));
      }
    });
  }
}

```

Using this approach, we can read user input as an `Observable` and create other fields using `RxObservableField`. For example,

```java
public final ObservableField<String> inputText = new ObservableField<>("");
public final RxObservableField<Boolean> errorVisible = new RxObservableField<>(toObservable(inputText).map(text -> text.isEmpty()))

```

Of course, if inputText is required to be updated based on some other interaction, it would have to be done imperatively inside the ViewModel by calling `set()`. This is a limitation which needs to be addressed. However, this approach works for now.


## RxObservableField > ReadOnlyField

In order to ensure consistency between the field value and observable source, we'll have to disable the setter causing the field to behave like a `ReadOnlyField`. Hence, it would be appropriate to rename `RxObservableField` to `ReadOnlyField`.


## Memory Leaks

In order to prevent memory leaks, it is necessary to clean up subscriptions. Because subscription is retained only when a view is observing the field, ensuring that view removes observer will ensure that subscription will not get retained. We can depend on the Data Binding layer for this.

Callbacks added to `ObservableField`, store weak reference to `ViewDataBinding`. As a result, uncleaned callbacks will not prevent `ViewDataBinding` from getting garbage collected. Also, `ViewDataBinding` removes all listeners before getting deallocated in `finalize()` method. Hence, on next GC after View is destroyed, all subscriptions in `ReadOnlyField` would also get removed.

## Source

These utilities are now available as part of my MVVM library. Check [manas-chaudhari/android-mvvm](https://github.com/manas-chaudhari/android-mvvm) Github project for the source, examples and documentation.
