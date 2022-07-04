# Wayland Core Protocol is tailored only for GNOME needs and the idea of protocol extensions doesn't work. (Opinion)

##### This article assumes that you know what's a [display server](https://itsfoss.com/display-server/). 

Wayland is a display server system based on the idea of [protocols](https://wayland.freedesktop.org/docs/html/ch04.html). That means that there is no Wayland display server that clients need to talk to. Instead, Wayland defines a protocol for creating display servers. Any client application that is programmed to use this protocol can work on any display server(or compositor) that fully supports supports this protocol. It's like the [world wide web protocols](https://www.w3.org/standards/), where any website is able to work on any browser as much the browser is completely supporting the web protocols. So you don't create websites for specific browsers.

That also means that the functions defined in the protocol decide what applications (aka: clients) can do & what they can't do. Returning to websites example, if the protocol doesn't define necessary functions, it will limit the web developers. Take CSS as an example, if it wasn't available in the web protocols, all websites would have looked the same and boring. So the protocol must include all necessary basics in a way that doesn't limit developers to few cases and uses.

When Wayland developers started defining the protocol, they had to decide what functionalities to include in the protocol. Their decision was to [make the protocol as minimal as possible](), and compositors shall create new protocols for their specific use cases if they desire to offer more functionality not included in the main protocol. The main protocol was called [Wayland Core protocol](https://wayland.app/protocols/wayland), and other protocols are called [protocol extensions](https://wayland.app/protocols/). All compositors are expected to support the core protocol, and they may not other protocol extensions. That means that applications which depend on certain functionality defined in one of the protocol extensions will not work on all compositors.

All of the above is what Wayland developers intended for the Wayland world to be. Now let's delve into more detail. How much is Wayland core protocol minimal? In other words, what determines what shall be in the core protocol and what shall not be? In this article I'm going to give you an answer to this question based on my opinion, which is in turn based on a group of simple facts.

My opinion is: Wayland's Core protocol is tailored only for GNOME needs. I mean that the functionalities which exist in Wayland's core protocol is the bare minimum required for GNOME desktop and apps to work on Wayland.
That's bad (still in my opinion) because it's simply not enough for other desktop environments and apps other than GNOME desktop and apps, as I'm going to show you in this article.

# Three facts that indicate that Wayland Core protocol is tailored for GNOME.

## 1. The core protocol requires that desktop visual components be drawn by the compositor.
First, let's explain something. On most desktop environments desktop components (like dock, panel, wallpaper, desktop icons, etc.) are regular clients. In order for those components to work, they need certain functions to be implemented by the compositor, those functions include:
    - Ability to move the window
    - Ability to tell the compositor to not draw decorations around said windows.
    -Ability to keep it above all windows(in case of the panel) or keep below all windows (in case of the background).
    - In addition to some other functionalities.
    
On X11, those were defined in what is called the [ICCCM specification](https://x.org/releases/X11R7.6/doc/xorg-docs/specs/ICCCM/icccm.html), which allows X11 clients to tell the compositor to do any of the above. On Wayland, there is not anything in the core protocol that allows that. Which means that desktop environments creators have to draw all these in the compositor. GNOME is the only desktop which does that, while many other desktops (KDE, XFCE, Lxqt, etc.) draw their components outside the compositor (an exception to that is cinnamon, because it started as a fork of GNOME 3). The situation is even worse. Apps like [plank dock], [latte dock] and other independent desktop  components can't exist in Wayland. There are protocol extensions that fix that, and I will talk about them later.

In summary the situation is:
    - Desktop environments have to draw everything in the compositor.
    - It's impossible to create cross-desktop desktop components like plak and latte dock
    
## 2. CSD is implementable although clients can't move their window.
We have known before that the core protocol doesn't define a way for clients to move their windows. So how is CSD implemented? Well, there is a [function in the protocol] that tells the compositor to start dragging the window. So instead of having a function for moving the window, which would had been useful in many cases they resorted to having a function only useful for implementing CSD.

## 3. CSD is a must in Wayland core protocol.
On X11, the situation was that apps expect to get decorated by the compositor (which is SSD) and if they wish to decorate themselves (by CSD) they tell the compositor to not draw its decorations. On Wayland, compositors are free to draw their decorations if they wishes. The problem is that there is not a way (inside the core protocol) for apps to know whether they are being decorated or not. In other words clients can't tell the compositor whether they prefer CSD or SSD. Which is problematic for both CSD and SSD (in theory). But in practice, GNOME decided to not decorate clients at all. So apps have to assume that they are not decorated due to GNOME's decision, so they must go for CSD. Again, there is a protocol extension that fixes that, more on that later.

## Summary of what has been mentioned so far.
The above three facts regarding the core protocol in summary are:
    1. Desktop components need to be drawn by the compositor
    2. CSD is a must.
    3. CSD is implementable although clients can't move their windows.

According to these 3 facts, I've concluded my opinion which is: Wayland's core protocol is tailored only for GNOME needs.

What if you wanted some functionalities not available in the core protocol. Wayland or GNOME developers answer to this is Wayland's protocol extensions. That simply means that compositors can offer extra functionality through creating new protocols. The problem in this approach is that means that some apps may work on some compositors and may not work on the rest of compositors (that's if it needs some of the protocol extensions). That may had resulted in severe fragmentation in theory, but the reality is less worse, that's due to the efforts of [wlroots project] and KDE.

# Wlroots has mostly saved the situation
[Wlroots](https://gitlab.freedesktop.org/wlroots/wlroots) is a library created by [Sway compositor](https://swaywm.org/) developers. It enables developers to create Wayland/X11 compositors easily. Their main focus is Wayland. There are already many compositors available based on wlroots. What is interesting though is the protocol extensions that wlroots implement.

Wlroots has many protocol extensions, including:
    - [LayerShell]() protocol
    - [xdg-decoration]() protocol

The LayerShell protocol enables desktop components to be drawn outside the compositor. Which also makes it possible to create independent cross-desktop desktop components. There are many projects that utilize this protocol that you can explore in the following repositories:
    -[sway shell]()
    -[wf-shell]()
    -[awesome-wayland]()

Also have a look at [GtkLayerShell library](). Which is a library for writing Gtk apps with LayerShell protocol.
Because LayerShell protocol is not a part of the core protocol apps using it work on wlroots based compositors and KDE, it's not supported only on GNOME.

The second protocol is xdg-decoration protocol.
Made by wlroots and KDE, it enables apps to choose between CSD and SSD.

These protocols work on wlroots compositor, and KDE. The only obstacle preventing the unification of Linux desktop is GNOME. They have decided to not implement any of these protocol extensions. Which put all apps that use SSD in a situation where they have to use SSD in supporting environments and CSD in gnome. The people actually feeling the pain are toolkits developers. To give you more context, have a look at the ["CSD initiative"](https://blogs.gnome.org/tbernard/2018/01/26/csd-initiative/) started by Tobias Bernard from GNOME, and [this blog post](https://blog.martin-graesslin.com/blog/2018/01/server-side-decorations-and-wayland/) from Martin's blog (kwin's developer). Also have a look at this [issue](https://gitlab.gnome.org/GNOME/mutter/-/issues/217). The situation is mostly solved by now, that Qt and Gtk draw CSD always on GNOME and utilize the xdg-decoration on other environments. However, in my opinion that is not good, because it makes the platform less standardized/unified for no good reason, because in the future, toolkits developers may decide to just go for CSD to avoid the pain.

# The root of all these problems.
In my opinion, the root of all these is GNOME or Wayland developers' decision to have the core as minimum as possible and require the creation of protocol extensions.
Imagine if the web worked in a similar way. That means that websites would not be able to target the standardized (minimal) protocols because they are not enough, and will rely on protocols created by certain browsers. So websites would target specific browsers, not the core protocol. That would had been a nightmare, right? Well that's the current Wayland situation.

# What is the solution?
The solution in my opinion, is to put all these protocol extensions in the core protocol, or that might not be necessary if GNOME implements the above protocols (Which is not likely to happen anytime soon.)
In simple words GNOME are the bottleneck of the problem, and they have the ability to solve it of they decide to do so.

