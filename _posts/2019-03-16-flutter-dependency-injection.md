---
layout: post
title:  "Dependency Injection in Flutter"
date:   2019-03-10 11:00:00 +0200
categories: flutter
---

As I wrote in a [previous article](https://developers.mewssystems.com/multiplatform-apps-are-we-there-yet/), we’re experimenting with Flutter while developing our side project for step challenges with colleagues. This side project should also be considered as a playground, where we can check if we can use Flutter in more serious projects. That’s why we want to use some approaches there that can look like an over-engineering for such a small project.

So one of the first questions was what can we use for dependency injection. A quick search in the internet revealed 2 libraries with positive reviews: [get_it](https://pub.dartlang.org/packages/get_it) and [kiwi](https://pub.dartlang.org/packages/kiwi). As `get_it` turned out to be a Service Locator (and I’m not a fan of this pattern), I was going to play with kiwi, which looked more promising, but then I’ve found another one library: [inject.dart](https://github.com/google/inject.dart). It is heavily inspired by Dagger library, and as we use the latest one in our other Android projects, I’ve decided to dig into it.

It’s worth saying that though this library is located in Google GitHub repository, it’s not an official library from Google and no support is currently provided:


> This library is currently offered as-is (developer preview) as it is open-sourced from an internal repository inside Google. As such we are not able to act on bugs or feature requests at this time.

Nevertheless it looks like the library does everything that we need for now, so I’d like to share some info on how you can use this library in your project.

## Installation

As there’s no package in [official repository](https://pub.dartlang.org/), we have to install it manually. I prefer to do it as a git submodule, so I’m creating a folder `vendor` in my project source directory and running the following command from this directory:

```sh
git submodule add https://github.com/google/inject.dart
```

And now we can set it up by adding following lines into `pubspec.yaml`:

```yaml
dependencies:
  # other dependencies here
  inject:
    path: ./vendor/inject.dart/package/inject

dev_dependencies:
  # other dev_dependencies here
  build_runner: ^1.0.0
  inject_generator:
    path: ./vendor/inject.dart/package/inject_generator
```

## Usage

What functionality do we usually expect from a DI library? Let’s go through some common use cases:

### Concrete class injection
It can be as simple as this:

```dart
import 'package:inject/inject.dart';

@provide
class StepService {
  // implementation
}
```

We can use it e.g. with Flutter widgets like this:

```dart
@provide
class SomeWidget extends StatelessWidget {
  final StepService _service;

  SomeWidget(this._service);
}
```

### Interface injection
First of all we need to define an abstract class with some implementation class, e.g.:

```dart
abstract class UserRepository {
  Future<List<User>> allUsers();
}

class FirestoreUserRepository implements UserRepository {
  @override
  Future<List<User>> allUsers() {
    // implementation
  }
}
```

And now we can provide dependencies in our module:

```dart
import 'package:inject/inject.dart';

@module
class UsersServices {
  @provide
  UserRepository userRepository() => FirestoreUserRepository();
}
```

### Providers
What to do if we need not an instance of some class to be injected, but rather a provider, that will give us a new instance of this class each time? Or if we need to resolve the dependency lazily instead of getting concrete instance in constructor? I didn’t find it neither in documentation (well, because there’s no documentation at all) nor in provided examples, but it actually works this way that you can request a function returning the required instance and it will be injected properly.

We can even define a helper type like this:

```dart
typedef Provider<T> = T Function();
```

and use it in our classes:

```dart
@provide
class SomeWidget extends StatelessWidget {
  final Provider<StepService> _service;

  SomeWidget(this._service);

  void _someFunction() {
    final service = _service();
    // use service
  }
}
```

### Assisted injection
There’s no built in functionality to inject objects that require arguments known at runtime only, so we can use the common pattern with factories in this case: create a factory class that takes all the compile-time dependencies in constructor and inject it,  and provide a factory method with runtime argument that will create a required instance.

### Singletons, qualifiers and asynchronous injection
Yes, the library supports all of this. There’s actually a good explanation in the [official example](https://github.com/google/inject.dart/blob/master/example/coffee/lib/src/drip_coffee_module.dart).

### Wiring it up
The final step in order to make everything work is to create an injector (aka component from Dagger), e.g. like this:

```dart
import 'main.inject.dart' as g;

@Injector(const [UsersServices, DateResultsServices])
abstract class Main {
  @provide
  MyApp get app;
  static Future<Main> create(
    UsersServices usersModule,
    DateResultsServices dateResultsModule,
  ) async {
    return await g.Main$Injector.create(
      usersModule,
      dateResultsModule,
    );
  }
}
```

Here `UserServices` and `DateResultsServices` are previously defined modules, `MyApp` is the root widget of our application, and `main.inject.dart` is an auto-generated file (more on this later).

Now we can define our main function like this:

```dart
void main() async {
  var container = await Main.create(
    UsersServices(),
    DateResultsServices(),
  );
  runApp(container.app);
}
```

## Running

As `inject` works with code generation, we need to use build runner to generate the required code. We can use this command:

```sh
flutter packages pub run build_runner build
```

or `watch` command in order to keep the source code synced automatically:

```sh
flutter packages pub run build_runner watch
```

But there’s one important moment here: by default the code will be generated into the `cache` folder and Flutter doesn’t currently support this (though there’s a work in progress in order to solve this problem). So we need to add the file `inject_generator.build.yaml` with the following content:


```yaml
builders:
  inject_generator:
    target: ":inject_generator"
    import: "package:inject_generator/inject_generator.dart"
    builder_factories:
      - "summarizeBuilder"
      - "generateBuilder"
    build_extensions:
      ".dart":
        - ".inject.summary"
        - ".inject.dart"
    auto_apply: dependents
    build_to: source
```

It’s actually the same content as in file `vendor/inject.dart/package/inject_generator/build.yaml` except for one line: `build_to: cache` has been replaced with `build_to: source`.

Now we can run the `build_runner`, it will generate the required code (and provide error messages if some dependencies cannot be resolved) and after that we can run Flutter build as usual.

## Profit

That’s all. You should also check the [examples](https://github.com/google/inject.dart/tree/master/example) provided with the library itself, and if you have some experience with Dagger library, `inject` will be really very familiar to you.

> Originally published at [developers.mews.com](https://developers.mews.com/dependency-injection-in-flutter/)
