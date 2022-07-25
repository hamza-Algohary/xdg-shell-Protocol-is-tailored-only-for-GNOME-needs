# Wayland xdg-shell Protocol is tailored only for GNOME needs. (Opinion)

##### This article assumes that you know what's a [display server](https://itsfoss.com/display-server/). 

Wayland is a display server system based on the idea of [protocols](https://wayland.freedesktop.org/docs/html/ch04.html). That means that there is no Wayland display server that clients need to talk to. Instead, Wayland defines a protocol for creating display servers. Any client application that is programmed to use this protocol can work on any display server(or compositor) that fully supports this protocol. It's like the [world wide web protocols](https://www.w3.org/standards/), where any website can work on any browser as much as the browser is completely supporting the web protocols. So you don't create websites for specific browsers.

That also means that the functions defined in the protocol decide what applications (aka clients) can do & what they can't do. Returning to the website's example, if the protocol doesn't define necessary functions, it will limit the web developers. Take CSS as an example, if it wasn't available in the web protocols, all websites would have looked the same and boring. So the protocol must include all necessary basics in a way that doesn't limit developers to few cases and uses.

When Wayland developers started defining the protocol, they had to decide what functionalities to include in the protocol. They decided to make the protocol as minimal as possible, and compositors shall create new protocols for their specific use cases if they desire to offer more functionality not included in the main protocol. The main protocol was called [Wayland Core protocol](https://wayland.app/protocols/wayland), and other protocols are called [protocol extensions](https://wayland.app/protocols/). All compositors are expected to support the core protocol, and they may not support other protocol extensions. That means that applications that depend on certain functionality defined in one of the protocol extensions will not work on all compositors.

The point of having Wayland's core protocol as minimal as possible is to allow the usage of Wayland in places other than desktop like embedded systems and VR. So Wayland's core protocol is not enough to even create desktop applications. That's why we have a protocol extension for desktop called xdg-shell. 

My opinion is that this protocol is tailored for gnome needs. I mean that the functionalities which exist in the xdg-shell protocol are the bare minimum required for GNOME desktop and apps to work on Wayland, as I am going to show you in this article.
That's bad (still in my opinion) because it's simply not enough for other desktop environments and apps other than GNOME desktop and apps, as I'm going to show you in this article.

# Three facts that indicate that xdg-shell protocol is tailored for GNOME.

## 1. The xdg-shell protocol requires that desktop visual components be drawn by the compositor.
First, let's explain something. On most desktop environments desktop components (like dock, panel, wallpaper, desktop icons, etc.) are regular clients. For those components to work, they need certain functions to be implemented by the compositor, those functions include:
* Ability to move the window
* Ability to tell the compositor to not draw decorations around said windows.
* Ability to keep it above all windows(in case of the panel) or keep it below all windows (in case of the background).
* In addition to some other functionalities.
    
On X11, those were defined in what is called the [ICCCM specification](https://x.org/releases/X11R7.6/doc/xorg-docs/specs/ICCCM/icccm.html), which allows X11 clients to tell the compositor to do any of the above. On Wayland, there is not anything in the xdg-shell protocol that allows that. This means that desktop environments creators have to draw all these in the compositor. GNOME is the only desktop which does that, while many other desktops (KDE, XFCE, Lxqt, etc.) draw their components outside the compositor (an exception to that is cinnamon because it started as a fork of GNOME 3). You might be wondering that such functionality is not necessary and that individual desktop environments can just implement their components by other protocol extensions. You might be right, but what about independent cross-desktop components? They are simply not implementable, apps like [plank dock](https://github.com/ricotz/plank), [latte dock](https://github.com/KDE/latte-dock), and other independent desktop  components can't be cross desktop anymore.

In summary, the situation is:
- Desktop environments have to draw everything in the compositor.
- It's impossible to create cross-desktop desktop components like plank and latte dock.
    
## 2. CSD is implementable although clients can't move their window.
We have known before that the xdg-shell protocol doesn't define a way for clients to move their windows. So how is CSD implemented? Well, there is a [function in the protocol](https://wayland-book.com/xdg-shell-in-depth/interactive.html) that tells the compositor to start dragging the window. So instead of having a function for moving the window, which would had been useful in many cases, they resorted to having a function only useful for implementing CSD.

## 3. CSD is a must in xdg-shell protocol.
On X11, the situation was that apps expect to get decorated by the compositor (which is SSD) and if they wish to decorate themselves (by CSD) they tell the compositor to not draw its decorations. On Wayland, compositors are free to draw their decorations if they wish. The problem is that there is no way (inside the xdg-shell protocol) for apps to know whether they are being decorated or not. In other words, clients can't tell the compositor whether they prefer CSD or SSD. Which is problematic for both CSD and SSD (in theory). But in practice, GNOME decided to not decorate clients at all. So apps have to assume that they are not decorated due to GNOME's decision, so they must go for CSD. There is a protocol extension that fixes that, more on that later.

## Summary of what has been mentioned so far.
The above three facts regarding the xdg-shell protocol in summary are:
1. Desktop components need to be drawn by the compositor
2. CSD is a must.
3. CSD is implementable although clients can't move their windows.

According to these 3 facts, I've concluded my opinion which is that Wayland's xdg-shell protocol is tailored only for GNOME needs.

What if you wanted some functionalities not available in the xdg-shell protocol. Wayland or GNOME developers answer to this is Wayland's protocol extensions. That simply means that compositors can offer extra functionality by creating new protocols. The problem with this approach is that means that some apps depending on those extra protocols will work on some compositors and will not work on the rest of the compositors. That may had resulted in severe fragmentation in theory, but the reality is less worse, that's due to the efforts of [wlroots project](https://gitlab.freedesktop.org/wlroots/wlroots) and KDE.

# Wlroots has mostly saved the situation
[Wlroots](https://gitlab.freedesktop.org/wlroots/wlroots) is a library created by [Sway compositor](https://swaywm.org/) developers. It enables developers to create Wayland/X11 compositors easily. Their main focus is Wayland. There are already many compositors available based on Wlroots. What is interesting though is the protocol extensions that Wlroots implement.

Wlroots has many protocol extensions, including:
- [LayerShell](https://wayland.app/protocols/wlr-layer-shell-unstable-v1) protocol
- [xdg-decoration](https://wayland.app/protocols/xdg-decoration-unstable-v1) protocol

The LayerShell protocol enables desktop components to be drawn outside the compositor. Which also makes it possible to create independent cross-desktop desktop components. There are many projects that utilize this protocol that you can explore in the following repositories:
- [nwg-shell](https://github.com/nwg-piotr/nwg-shell)
- [wf-shell](https://github.com/WayfireWM/wf-shell)
- [awesome-wayland](https://github.com/natpen/awesome-wayland)

Also, have a look at [GtkLayerShell library](https://github.com/wmww/gtk-layer-shell). Which is a library for writing Gtk apps with LayerShell protocol.
Because LayerShell protocol is not a part of the xdg-shell protocol apps using it work on Wlroots-based compositors and KDE, it's not supported only on GNOME.

The second protocol is xdg-decoration protocol.
Made by wlroots and KDE, it enables apps to choose between CSD and SSD.

These protocols work on wlroots compositor, and KDE. The only obstacle preventing the unification of Linux desktop is GNOME. They have decided to not implement any of these protocol extensions. Which put all apps that use SSD in a situation where they have to use SSD in supporting environments and CSD in gnome. The people actually feeling the pain are toolkits developers. To give you more context, have a look at the ["CSD initiative"](https://blogs.gnome.org/tbernard/2018/01/26/csd-initiative/) started by Tobias Bernard from GNOME, and [this blog post](https://blog.martin-graesslin.com/blog/2018/01/server-side-decorations-and-wayland/) from Martin's blog (kwin's developer). Also, have a look at this [issue](https://gitlab.gnome.org/GNOME/mutter/-/issues/217). The situation is mostly solved by now, that Qt and Gtk draw CSD always on GNOME and utilize the xdg-decoration on other environments. However, in my opinion that is not good, because it makes the platform less standardized/unified for no good reason, because in the future, toolkits developers may decide to just go for CSD to avoid the pain.

By the way, what I mean by saying that wlroots has saved the situation is that without their efforts we would not have had these features which make Wayland more usable. So the only thing remaining is standardizing these protocols, and that will only happen if GNOME implemented these protocol extensions.

# Why does GNOME insist on not implementing other protocol extensions?
I can't see any reason for GNOME not implementing these protocols except that they don't want to write and maintain more code. Apart from Wayland, it's a pattern I see that's always influencing their design decisions. Many of the features which were removed from GNOME like desktop icons and system tray was because their code was complicated so they just removed them. If you like me prefer CSD, then you might think that GNOME's decision might make all apps go CSD, but that is not the reality, because CSD just means that clients draw their own decorations, it does not necessarily mean that titlebars will start containing buttons automatically. If you try to create a hello world program using any GUI toolkit you will first create an empty window and you might also leave the titlebar empty, so titlebars have to be empty by default so forcing CSD is not going to force a design, contrary to what many people think. So, in the end, we can't say that GNOME wants to force CSD because they like its look, but because they simply don't want to write more code. A desktop environment being opinionated is not bad, but the entire platform being opinionated is a nightmare for other desktop environments and people who will write desktop environments or toolkits in the future.

Whatever you think regarding these decisions, I believe that in the future we must head for more unification. Otherwise, we will be having a terribly fragmented platform.
