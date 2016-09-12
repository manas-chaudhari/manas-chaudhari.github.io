---
layout: "post"
title: "RxJava meets Data Binding IV - Farewell Subscriptions"
date: "2016-09-05 01:00"
comments: true
tags:
- android-mvvm
---

A subscription free way of using RxJava.


RxJava provides an easy way to model changing data as an `Observable`. To consume the data, it is required to `subscribe` to the `Observable`, which generates a `Subscription` which needs to be unsubscribed later (in most cases) to prevent memory leaks. As long as we are manipulating Observables (Rx world), the code is functional and all is well. However, subscribing and unsubscribing operations which are dependent on Android lifecycle, need to be done in the imperative world and result in boilerplate code.

Most existing techniques recommend subscribing to the Observable and cleaning up the subscription object in `onDestroy`. [RxLifecycle](https://github.com/trello/RxLifecycle) library provides a convenient way to implement this by making use of `takeUntil` operator. However, this approach still requires you to use `.compose(RxLifecycle.bindUntilEvent(lifecycle, ActivityEvent.DESTROY))` on each observable before subscription. I would like to improve this by eliminating the subscription itself.


### Starter Code

Chris Arriola recently wrote an [Introduction to RxJava](https://www.toptal.com/android/functional-reactive-android-rxjava) in which he builds an example app which fetches data from Github API and displays in a ListView. I have taken his code as the starting point. Initial code is available [here](https://github.com/manas-chaudhari/GitHubRxJava/tree/solution)

### Setup library
I'll be using my [Android MVVM](https://github.com/manas-chaudhari/android-mvvm) which provides tools to implement MVVM using RxJava and Data Binding.

In build.gradle, add the dependency & enable data binding

```groovy
android {
  ...

  dataBinding {
    enabled = true
  }
}

dependencies {
  compile 'com.manaschaudhari:android-mvvm:0.1.2'
}
```

Create `GithubApplication.java` to initialize the library.

```java
public class GithubApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        BindingUtils.setDefaultBinder(new ViewModelBinder() {
            @Override
            public void bind(ViewDataBinding viewDataBinding, ViewModel viewModel) {
                viewDataBinding.setVariable(BR.vm, viewModel);
            }
        });
    }
}
```

### Create ViewModels

Move presentation logic [GithubRepoAdapter](https://github.com/manas-chaudhari/GitHubRxJava/blob/solution/app/src/main/java/com/chrisarriola/githubrxjava/GitHubRepoAdapter.java) to `GithubRepoVM`.

```java
public class GithubRepoVM implements ViewModel {
    public final String name;
    public final String description;
    public final String language;
    public final String stars;

    public GithubRepoVM(GitHubRepo repo) {
        name = repo.name;
        description = repo.description;
        language = "Language: " + repo.language;
        stars = "Stars: " + repo.stargazersCount;
    }
}
```

The logic in [MainActivity](https://github.com/manas-chaudhari/GitHubRxJava/blob/solution/app/src/main/java/com/chrisarriola/githubrxjava/MainActivity.java) uses an imperative style as it subscribes to the observable on click. This can be improved by getting username text & click event as Observables and combining them using [Sample](http://reactivex.io/documentation/operators/sample.html) operator. `MainViewModel` then looks as follows:

```java
import static com.manaschaudhari.android_mvvm.FieldUtils.toObservable;

public class MainViewModel implements ViewModel {
    public final Observable<List<GithubRepoVM>> repositories;
    public final ObservableField<String> username = new ObservableField<>();
    public final PublishSubject<Void> onSearchClick = PublishSubject.create();

    public MainViewModel() {
        repositories =
                toObservable(this.username)
                .sample(onSearchClick)
                .switchMap(new Func1<String, Observable<List<GithubRepoVM>>>() {
            @Override
            public Observable<List<GithubRepoVM>> call(String username) {
                return GitHubClient.getInstance()
                        .getStarredRepos(username)
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .map(new Func1<List<GitHubRepo>, List<GithubRepoVM>>() {
                            @Override
                            public List<GithubRepoVM> call(List<GitHubRepo> gitHubRepos) {
                                List<GithubRepoVM> vms = new ArrayList<>();
                                for (GitHubRepo repo : gitHubRepos) {
                                    vms.add(new GithubRepoVM(repo));
                                }
                                return vms;
                            }
                        });
            }
        });
    }
}}
```

### Update `item_github_repo.xml` to use Data Binding

```diff
<?xml version="1.0" encoding="utf-8"?>
+ <layout xmlns:android="http://schemas.android.com/apk/res/android"
+     xmlns:tools="http://schemas.android.com/tools">
+
+     <data>
+
+         <variable
+             name="vm"
+             type="com.chrisarriola.githubrxjava.GithubRepoVM" />
+     </data>

      <RelativeLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:orientation="vertical"
          android:padding="6dp">

          <TextView
              android:id="@+id/text_repo_name"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:textSize="24sp"
              android:textStyle="bold"
+             tools:text="@{vm.name}" />

          <TextView
              android:id="@+id/text_repo_description"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:layout_below="@+id/text_repo_name"
              android:ellipsize="end"
              android:lines="2"
              android:textSize="16sp"
+             tools:text="@{vm.description}" />

          <TextView
              android:id="@+id/text_language"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_alignParentLeft="true"
              android:layout_below="@+id/text_repo_description"
              android:textColor="?attr/colorPrimary"
              android:textSize="14sp"
              android:textStyle="bold"
+             tools:text="@{vm.language}" />

          <TextView
              android:id="@+id/text_stars"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_alignParentRight="true"
              android:layout_below="@+id/text_repo_description"
              android:textColor="?attr/colorAccent"
              android:textSize="14sp"
              android:textStyle="bold"
+             tools:text="@{vm.stars}" />

      </RelativeLayout>
+ </layout>
```

### Update `activity_main.xml` to use Data Binding
Note that I have used BindingAdapters provided by the library for setting up the RecyclerView.

```diff
<?xml version="1.0" encoding="utf-8"?>
+ <layout xmlns:android="http://schemas.android.com/apk/res/android"
+     xmlns:bind="http://schemas.android.com/apk/res-auto">
+
+     <data>
+
+         <variable
+             name="vm"
+             type="com.chrisarriola.githubrxjava.MainViewModel" />
+     </data>

      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:orientation="vertical">

+         <android.support.v7.widget.RecyclerView
              android:id="@+id/list_view_repos"
              android:layout_width="match_parent"
              android:layout_height="0dp"
              android:layout_weight="1"
+             bind:items="@{vm.repositories}"
+             bind:layout_vertical="@{true}"
+             bind:view_provider="@{@layout/item_github_repo}" />

          <LinearLayout
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:orientation="horizontal">

              <EditText
                  android:id="@+id/edit_text_username"
                  android:layout_width="0dp"
                  android:layout_height="wrap_content"
                  android:layout_weight="1"
                  android:hint="@string/username"
+                 android:text="@={vm.username}" />

              <Button
                  android:id="@+id/button_search"
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
+                 android:onClick="@{vm.onSearchClick}"
                  android:text="@string/search" />

          </LinearLayout>

      </LinearLayout>
+ </layout>
```

#### Binding click events to PublishSubject

For searchClick event, we used a `PublishSubject` in `MainViewModel`. Binding can be implemented using a `BindingConversion` which converts the subject to `OnClickListener`.

```java
@SuppressWarnings("unused")
public class BindingAdapters {

    @BindingConversion
    public static View.OnClickListener subjectToListener(final PublishSubject<Void> subject) {
        if (subject != null) {
            return new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    subject.onNext(null);
                }
            };
        } else {
            return null;
        }
    }
}
```

### Cleanup MainActivity and Delete GithubRepoAdapter

As the library provides a generic adapter for RecyclerView, there is no need to write custom adapters. Thus, `GithubRepoAdapter` can be removed.

As we have moved all the required logic from activity to ViewModels, we can delete the rest of the code. All that remains is specifying the ViewModel and View.

```java
public class MainActivity extends MvvmActivity {

    @NonNull
    @Override
    public ViewModel createViewModel() {
        return new MainViewModel();
    }

    @Override
    public int getLayoutId() {
        return R.layout.activity_main;
    }
}
```

### Summary

- The library takes care of subscribing to observables and cleaning up. Thus, code is free of memory leaks by default.
- Presentation logic has been moved from Activity/Adapters to ViewModels. This allows for better reuse.
- Boilerplate removed for
    - Subscribing and cleaning subscriptions
    - `findViewById`
    - setting up RecyclerView

Checkout the diff from initial point to final point: [https://github.com/manas-chaudhari/GitHubRxJava/pull/1/files?w=1](https://github.com/manas-chaudhari/GitHubRxJava/pull/1/files?w=1)

The Android MVVM library is available [here](https://github.com/manas-chaudhari/android-mvvm). I highly recommend going through the [README](https://github.com/manas-chaudhari/android-mvvm) to get an idea about what's happening under the hood.

### Source
The source is available at:

- [Diff](https://github.com/manas-chaudhari/GitHubRxJava/pull/1/files?w=1)
- [File Tree](https://github.com/manas-chaudhari/GitHubRxJava/tree/solution-mvvm)
- [Android MVVM](https://github.com/manas-chaudhari/android-mvvm)
