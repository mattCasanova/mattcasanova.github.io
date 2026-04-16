---
layout: post
title: "House Rules: I Know What the Other Four Letters Are"
series: house-rules
tags: [architecture, solid, mobile, ios, swift, opinion]
---

A few jobs ago, my team had one of those team-event nights — drinks at a bar after work, the kind where half the team orders water and the other half pretends they're going to have "just one." I was probably three or four drinks in. Maybe more — I forget.

I was definitely the only person at the table who'd committed to the evening.

At some point the conversation drifted to code architecture, because most of us are introverted nerds anyway. Myself included, obviously, since it's 11pm and I'm writing a blog post about the finer points of code responsibility. Somebody brought up SOLID. "You know, single responsibility and all that." Then he said the thing that stuck with me: *"Nobody really knows the other four."*

I rattled off all five without thinking about it. Single Responsibility. Open-Closed. Liskov Substitution. Interface Segregation. Dependency Inversion. Cold. In order. Six-ish drinks deep.

Not because I'm smarter than anybody at that table — that's certainly not true, there were at least three people with master's degrees sitting there that night. But because I spent seven years teaching this stuff to undergrads. When you explain the Liskov Substitution Principle to a room full of college sophomores twice a year for seven years, it stops being something you look up and starts being something you just *have*.

The guy was right, though. Not about me — about the industry. Most working engineers know the S. Maybe the D, vaguely. The middle three are a blur. And the irony is that I just spent [four posts](/series/bad-advice/) yelling about the D in SOLID — [Roll Your Own DI](/2026/04/14/bad-advice-roll-your-own-di/), [Delete Your Singletons](/2026/04/14/bad-advice-delete-your-singletons/), [Roll Your Own Fakes](/2026/04/14/bad-advice-roll-your-own-fakes/), [Wrap Your Vendors](/2026/04/15/bad-advice-wrap-your-vendors/) — the whole arc was the Dependency Inversion Principle dressed up in bar-stool clothes. I named it once or twice in passing. The rest of the time I just argued for it.

So let me back up. This is the first post in a new series called **House Rules** — not contrarian takes, just the principles I actually build on. Starting with S, because it's the one everybody *thinks* they know and almost nobody applies correctly.

## Four Phrases for the Same Damn Thing

There's a pile of architecture jargon that floats around code reviews and interview prep, and most of it is saying the same thing:

- **Single Responsibility Principle**
- **Separation of concerns**
- **High cohesion, low coupling**
- **One reason to change**

Four phrases. One idea. Most people know one or two of them and don't realize they're the same argument wearing different outfits.

They're all asking the same question: **when this code has to change, how much else breaks?** That's the whole game. SRP is where it starts.

## "Do One Thing" Is Not What the S Means

Here's the thing most people get wrong. They hear "single responsibility" and they think it means *"a class should do one thing."* That's close but imprecise, and the imprecision is where bad architecture sneaks in.

Uncle Bob's actual formulation is:

> **A class should have only one reason to change.**

Those aren't the same sentences. "Do one thing" is about *what the code does*. "One reason to change" is about *who can force this code to change*.

That difference matters.

Think about it this way. A `LoginViewModel` that checks whether the user typed something, calls an auth API, and updates UI state is "doing three things." But all three change for the same reason: the login flow changed. The designer wants a different error message? That's the login flow. The backend changes the auth endpoint? That's the login flow. The PM wants a loading spinner? Still the login flow. One reason to change, even though the class has multiple methods.

(Side note: my login view model doesn't validate email format at all. Laravel already does that on the backend, and it's better at it than any regex I'd write. The user typed something and pressed login? Good enough — let the server decide. Thin client. That's a different post, but it's worth mentioning because it's a tradeoff I made deliberately, not an oversight.)

Now think about a `UserManager` that handles authentication *and* profile image uploads *and* analytics events. How many things is it doing? Three, sure. But more importantly: **who asks for changes?** The security team asks for auth changes. The design team asks for image-upload changes. The growth team asks for analytics changes. Three different stakeholders. Three different reasons to change. Three responsibilities, even if the class "works fine" today.

The class doesn't know it has a problem until two of those stakeholders need a change in the same sprint and you're merging conflicts in a 600-line file that does too much.

**SRP isn't counting methods. It's counting stakeholders.**

## The 2015 iOS Problem

I've been building mobile apps since 2010 — mobile games, back when Angry Birds was the height of mobile sophistication — then production apps from around 2017. And I need to be honest about what the iOS world looked like when I started doing non-game mobile work.

Everything was in the `UIViewController`.

Business logic. API calls. Navigation. State management. Formatting. Validation. Animation. All of it, in one file, because that's what the tutorials showed and that's what the framework encouraged. Apple gave you a `UIViewController` with lifecycle hooks and said *"here, build your screen."* They didn't say where the business logic should go. They didn't give you a view model layer. They gave you Storyboards for UI layout, which worked fine for one developer and fell apart the moment you put the XML in Git with a team — merge conflicts in Storyboard files were legendary. So teams started writing UI in code too, and now the view controller had *everything*: layout, business logic, networking, and navigation, all in the same class.

Android had the same problem with its `Activity` class. Everything lived there because there was nowhere else to put it.

Nobody was shipping view models in 2015. Not nobody-nobody — MVVM existed as a pattern. But the default path the frameworks gave you was "put it in the controller," and most people followed the default path. And honestly, most of the smaller dev shops didn't have teams — they had one dev per platform, maybe one dev for *both* platforms. No code review. No second opinion. No one to say *"hey, should this really all be in the view controller?"* You wrote what the tutorials showed, you shipped it, and you moved on to the next screen.

The result was view controllers with 800, 1,200, sometimes 2,000 lines. Every feature request from every stakeholder landed in the same file. Auth changes? `LoginViewController`. New analytics event? `LoginViewController`. Different error formatting? `LoginViewController`. Profile image upload? You guessed it. One file. Five reasons to change. SRP violated at every level, but nobody called it that — they called it *"the app works."*

It worked until it didn't. And "didn't" usually looked like: two engineers trying to ship unrelated features in the same sprint, both touching the same 1,200-line file, both generating merge conflicts that took longer to resolve than the features themselves.

## The Code I Actually Ship

I'm building an iOS app as a side project right now, and I knew going in that I'd eventually want a tvOS version too. So I made a decision on day one: the business logic can't live in the app target. If it does, I'm rewriting it when the TV app comes. I've done that rewrite before. I'm not doing it again.

So I split the project into two pieces: a **core library** with all the business logic, and an **app target** that's nothing but views. The core library exports **protocols** (Swift's word for interfaces) and **view models**. The concrete service implementations — `DefaultAuthService`, `DefaultAPIService`, `DefaultLoggingService` — are all `internal`. The app target can't see them. Can't reach them. Doesn't know they exist.

Here's the `LoginViewModel`. The whole thing:

```swift
@Observable
@MainActor
public final class LoginViewModel {
    private let authService: AuthService

    public var email = ""
    public var password = ""
    public private(set) var isLoading = false
    public private(set) var error: ClientError?
    public private(set) var fieldErrors: [String: [String]] = [:]
    public private(set) var didLoginSuccessfully = false

    public init(authService: AuthService) {
        self.authService = authService
    }

    public func login() async {
        guard !email.trimmingCharacters(in: .whitespaces).isEmpty,
              !password.isEmpty
        else { return }

        isLoading = true
        error = nil
        fieldErrors = [:]

        do {
            _ = try await authService.login(email: email, password: password)
            didLoginSuccessfully = true
        } catch let APIError.validationError(_, errors) {
            fieldErrors = errors
        } catch APIError.networkError {
            error = .network
        } catch {
            self.error = .generic
        }

        isLoading = false
    }
}
```

Forty-four lines. It takes an `AuthService` *protocol* through its constructor. It holds form state. When you call `login()`, it calls the service and routes the result into observable properties. That's it. There is no networking code. No storage code. No analytics code. No navigation code.

Can you find the business logic? No? Good. It's in the service, where it belongs.

I know what you're going to say. *"Sure, but this isn't a real app — you're not doing anything with it."* That's not true. This is a fully-featured app with error logging, API calls, and a website to back it. Behind that one `authService.login()` call, here's what actually happens: the API service makes the real HTTP request. If it fails, the error gets logged to Sentry — through an interface. If it succeeds, the auth service stores the current user in local storage — through an interface — and saves the auth token in secure storage (the keychain, separate from regular storage) — through an interface. The `onLogin` hook fires, which sets the user context on the logger and refreshes feature flags. If the API returns validation errors, they bubble back up through the view model and land directly in the `FormField` components — the view doesn't even handle errors itself, it just passes `viewModel.fieldErrors["email"]` to the form field and the form field knows how to display them.

All of that is happening, and the login view model doesn't need to know about any of it. If you build it right from the start, complex apps look simple — because the complexity is distributed across the right layers instead of piled into one file.

And the blast radius? Re-theme the app? That's the theme file — the view model doesn't change. Add a button to the login screen? That's the view — the view model doesn't change. Swap Sentry for a different logger? That's one new implementation of `LoggingService` — nothing else in the entire app changes. Every change lands in one file. That's SRP paying rent.

And here's the view that consumes it:

```swift
struct LoginView: View {
    @Bindable var viewModel: LoginViewModel
    var onSignUpTapped: () -> Void
    var onForgotPasswordTapped: () -> Void

    var body: some View {
        ScrollView {
            VStack(spacing: AppTheme.Spacing.xl) {
                // ... title, subtitle ...

                FormField(
                    label: "Email",
                    text: $viewModel.email,
                    errors: viewModel.fieldErrors["email"],
                )

                FormField(
                    label: "Password",
                    text: $viewModel.password,
                    errors: viewModel.fieldErrors["password"],
                    isSecure: true,
                )

                Button(action: { Task { await viewModel.login() } }) {
                    if viewModel.isLoading {
                        ProgressView()
                    } else {
                        Text("Log In")
                    }
                }
            }
        }
    }
}
```

The view doesn't know what an API call is. It doesn't know what an `AuthService` is. It binds to `viewModel.email`. It binds to `viewModel.isLoading`. When the button is tapped, it calls `viewModel.login()`. The view's only job is:

Showing shit and reacting to shit.

That's it.

The frameworks finally caught up. It just took a decade of massively bloated view controllers to get there.

## Turtles All the Way Down

Not gonna brag, but it's not just the view model. It's SRP applied everywhere:

| Layer | Responsibility | Who asks for changes |
|-------|---------------|---------------------|
| **View** | Display UI, bind to state, forward taps | Design team / UI refresh |
| **ViewModel** | Coordinate UI state ↔ services | Product team / flow changes |
| **Service** | Execute domain logic (auth, storage) | Backend team / vendor changes |

Swap `UserDefaults` for Core Data? That's the storage service — the views and view models don't know and don't care. UI redesign? That's the view — the `AppContainer` and every service behind it stay untouched. A change in the view model might break some shit, sure — but it won't break the services, and it won't break a view on a different platform that binds to the same data.

That's the whole point of the two-target split. When I build the tvOS app, it gets its own views — bigger fonts, remote-control navigation, different layout — but it imports the same core library. Same view models, same services, same `AppContainer`. Not one line of business logic duplicated. The two apps change independently because they only share what *should* be shared.

## Where I Break My Own Rules

Here's my confession. I just spent four posts screaming about putting everything behind interfaces. You might have noticed that my view models are not interfaces.

WTF, right?

That's fair. I've done it that way before — view models behind protocols, mock view models for snapshot testing, all that stuff. If you can't mock it, you can't test it.

But right now my view models are thin. The `LoginViewModel` calls a service and routes the result into state. Testing it is easy — create mock services, pass them in, done.

The moment that changes — the moment the view models start getting a bunch of logic, crazy workflows, lots of navigation stuff, stuff that makes "just pass a fake service" feel like a pain — I'll switch them over to protocols. That's obviously the right call at that point, and it's not a hard refactor.

I'm not doing it now, because right now it's easy to test. That's the key. The moment it stops being easy to test, that's my signal.

Or maybe I'm just full of it.

Either way, that's really the discipline side of SRP — the people who get it wrong either split too early or too late. Too early is adding a protocol on a 44-line pass-through for theoretical future testability. Too late is letting a view controller grow to 1,200 lines because "it works and we're busy." The skill is the timing.

## Know When to Fold Them

So that's it. That's the Single Responsibility Principle.

You don't have to make it a day-one decision. But you should be constantly thinking about it. You watch. You notice when it starts feeling like there's more than one reason to change. And then you change it.

That 2015 `UIViewController` wasn't wrong on day one. On day one it was a prototype with one screen and one developer. It became wrong on day 90, when the second feature landed, and the third, and the fourth, and nobody stopped to ask *"who else can force this file to change?"*

The goal is always the same: **minimize the blast radius.** Every time I wrote that phrase in the Bad Advice series, I was writing about SRP without saying the letters. DI, singletons, fakes, vendor wrappers — all the same question. *When this has to change, how much else breaks?*

S is the foundation. The next four letters are the load-bearing walls.

I'll get to them.

---

*This is the first entry in the **House Rules** series — the principles I actually build on, explained with the code I actually ship. Next up: the O. Open-Closed Principle, and why "open for extension, closed for modification" is the most misunderstood sentence in software engineering.*

*The guy at the bar was right, by the way. Most people don't know the other four. But now you know the first one — the real version, not the bumper-sticker version. Four more to go.*

*I didn't write them, but those are the rules.*
