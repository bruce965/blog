---
title: "Native web apps without Electron"
date: 2024-01-07 16:33:44 +0100
categories: [Software]
tags: [Development, Web, HTML5]
img_path: /assets/img/posts/2024/
---

In the last few years, web technologies have evolved at a fast and steady pace,
and developers have started to like web-based interface more and more.

[Electron](https://www.electronjs.org/)-based application look and feel like
native applications, but behind the curtains there's a custom version of
Chromium, the open-source version of Google Chrome, rendering a web page.

Developers have started to build applications with Electron, thus being able to
use familiar web technologies such as HTML5, CSS and JavaScript (or maybe
[TypeScript](https://www.typescriptlang.org/)), as well as their trusted
frameworks such as [React](https://react.dev/) (my personal favourite),
[Vue](https://vuejs.org/) or [Angular](https://angular.io/).

Using Electron to build user interfaces not only has the huge benefit of letting
developers code with their favourite tools, but also makes it very easy to
_share the majority of the code_ when maintaining both a native app and a web
app _accessible through a web browser_, and since Electron is cross-platform,
it's usually possible to just write code once, building for all major operating
systems from a single codebase.

> Fun fact: [Visual Studio Code](https://code.visualstudio.com/) is actually an
> Electron application.\
> Ah yes, you can also [run Visual Studio Code in your browser](https://vscode.dev/).
> Neat!
{: .prompt-info }

Unfortunately, there are also a few drawbacks. The main issue with Electron is
that, being closely related to Google Chrome, it consumes quite a lot of RAM.
Another issue is that each app release includes a copy of Electron, which again,
is basically a custom version of Google Chrome, thus taking up quite a bit of
disk space.

Some alternatives emerged, such as [Tauri](https://tauri.app/), which still lets
developers write code with familiar web technologies, but bundling only what is
necessary to render a user interface instead of an entire web browser, and
relying on the low-level libraries that the operating system provides.


## The idea

So, expanding on Tauri's idea I was thinking... _why bundling anything at all_?
Almost all computers already have a web browser installed, what's the point of
including it in every deployment? Wouldn't it be possible to just develop our
applications using web technologies, and then let the user's web browser draw
the interface, while still looking like a native application? If this was
possible, we would be able to get all the benefits, while still mitigating
the drawbacks.

I am certainly not the first to have this idea, but I still want to explore it
in this blog post. The end goal is to build a minimal "to do list" application
_using web technologies_, which must _look and feel like a native application_,
and should _not include any layout engine or JavaScript interpreter_.


## Experimenting

### Chromium

So... let's assume the user has a Chromium-based browser installed, such as
Google Chrome, Microsoft Edge, Brave, Opera, Vivaldi, or Epic.

On modern Windows installations Microsoft Edge is pre-installed, so we can
expect to always find at least one Chromium-based browser on this platform.

How can we open a web page in a Chromium-based browser such as Microsoft Edge,
while making it appear like a native application?

```bash
msedge --new-window --app=https://example.com
```

And what does it look like on Windows 10?

![Microsoft Edge running https://example.com in "app mode" on Windows 10.](browser-chrome-app.png){: .shadow w='817' h='444' }

Not too bad. This looks pretty much like a regular window from a native app.

Unfortunately this doesn't fully work as expected. Running a Chromium-based
browser like this will include all custom user settings and extensions, and this
might interfere with our interface.

Fortunately, the solution is quite straightforward. Chromium-based browsers
allow specifying a path to store settings separately from the main installation.

Let's try running the following command from a Windows 10 command prompt:

```bash
msedge --new-window --app=https://example.com --user-data-dir="%AppData%/MyAppTest/profile-msedge"
```

This command will create a new a separate profile and store it in
_"%AppData%/MyAppTest/profile-msedge"_.

Great! This should be enough for Windows users.


### Firefox

Non-Windows users might not have a Chromium-based browser installed, but I
would assume most Linux users either have a Chromium-based browser or Firefox
installed.

How can we open a web page in Firefox, while making it appear like a native
application?

Unfortunately it's not as simple as with Chromium-based browsers, we have to
work a little harder to alter some of Firefox's default settings.

Let's create the template for a Firefox profile, with a _prefs.js_ file to
customize Firefox's behaviour, and a _userChrome.css_ file to customize
Firefox's appearance by running the following commands from a Linux terminal:

```bash
mkdir -p "$HOME/.my-app-test/profile-firefox"
cat <<EOF > "$HOME/.my-app-test/profile-firefox/prefs.js"
user_pref("browser.shell.checkDefaultBrowser", false);
user_pref("browser.tabs.warnOnClose", false);
user_pref("toolkit.legacyUserProfileCustomizations.stylesheets", true);
EOF

mkdir -p "$HOME/.my-app-test/profile-firefox/chrome"
cat <<EOF > "$HOME/.my-app-test/profile-firefox/chrome/userChrome.css"
#toolbar-menubar { height: 38px !important; }
#TabsToolbar { display: none; }
#menubar-items { display: none; }
#nav-bar { visibility: collapse; }
EOF
```

We can now run Firefox with the following command:

```bash
firefox -profile "$HOME/.my-app-test/profile-firefox" -no-remote -url "https://example.com"
```

And what does it look like?

![Firefox running https://example.com with custom appearance on Linux with Gnome.](browser-firefox-app.png){: .shadow w='817' h='453' }

It definitely still needs some polishing, as the window's title is not visible
and there are some other small issues. Still, it's a decent starting point!


### Safari

Some Mac users might only have Safari installed.

How can we open a web page in Safari, while making it appear like a native
application?

Unfortunately I don't have a Mac, so this question will have to remain
unanswered for now.


### Others

What if we are on another exotic installation, such as on a low-power device
such as a Raspberry Pi Zero, which comes with Midori as the default web browser?

For this and all other special cases we can still launch the default browser
without any special directives, it will not appear like a native application,
but it is guaranteed to always work as long as there is any web browser
installed.

On Linux, for instance, we can use the following command:

```bash
xdg-open "https://example.com"
```

Unfortunately there's nothing interesting to see here, the app will open in the
default browser, and therefore it will not be characterized by the distinctive
look of a native app.

![Chromium with the https://example.com open on Linux with Gnome.](browser-chrome-web.png){: .shadow w='817' h='498' }

Still, at least the user will be able to see something and interact with the app
even on systems where the interface has never been tested.


## Proof of Concept

Now that we know what is possible, it's time to build a small "to do list" app
in React, and bundle it into a native executable server app written in .NET 8.0.
I will probably attempt to do this in the future.

> TODO: perhaps someday...


## System Theme

It would be nice to use the operating system's theme to style components such as
input fields, buttons, checkboxes, etc... I'm still not sure this is entirely
possible, but perhaps I will give it a try in the future.

> TODO: perhaps someday...


## Extras

Thank you very much for taking the time to read my article, I hope you liked it!

If you found this experiment interesting, you might want to check out
[WebUI](https://github.com/webui-dev/webui), a little C library to do something
very similar to what we explored here.
