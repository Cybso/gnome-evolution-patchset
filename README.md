# Cybso's patchset for Evolution

Since my employer has migrated to Microsoft, I needed an e-mail client / calendar with good EWS support.

My previous favorite, the KDE client Kontact, unfortunately only offers read access to the EWS calendar and was therefore not an option.

There are several extensions for Thunderbird, but none of them really convinced me.

[Evolution](https://wiki.gnome.org/Apps/Evolution), on the other hand, offered me excellent EWS integration including calendar right from the start.

However, in some places Evolution does not behave the way I prefer to work. That's why I maintain my own private patch set, which I apply to the current stable Debian version of the package (currently version 3.46.4).

## The patches

### d21c5b43-use-monospace-in-markdown-3.46.4-2.patch

Enforces the usage of the configured monospace font in Evolution's markdown editor.

Backported from [issue #2654](https://gitlab.gnome.org/GNOME/evolution/-/issues/2654) to 3.46.4.

### disable-message-selection-fallback.patch

When using search folders Evolution auto-selects the first unread message from the list.

This has often led to me unintentionally setting unread e-mails to "read" and overlooking them as a result.

There was a partial fix in [issue #2289](https://gitlab.gnome.org/GNOME/evolution/-/issues/2289), but it did not cover all possible cases. This patch disables this fallback completely.

### markdown-dont-plain-quote-signature.patch

Evolution adds source quotes (``` ... ```) around signatures when in Markdown Editor mode. I just don't want this.

### send-after-1-minute.patch

Originally, Evolution only had the options of sending emails _immediately, _only manually_ or automatically _after 5 minutes_. After a lengthy discussion in [issue #2494](https://gitlab.gnome.org/GNOME/evolution/-/issues/2494), the option "Send after 1 minute" was added as a compromise. This patch is backported to 3.46.4.

### use-native-dialogs-which-are-compatible-with-portal.patch 

I am using Evolution in a Plasma (KDE) environment. With [KDE Portal for XDG Desktop](https://invent.kde.org/plasma/xdg-desktop-portal-kde) I can use Plasma's file chooser dialogs in GTK applications by settings the environment variable `GTK\_USE\_PORTAL=1`. However, I noticed that this doesn't work when adding attachments to an mail.

After having a look into the source I have noticed that Portal does not work when `gtk\_file\_chooser\_dialog\_new` is used. However, Evolution already contains a fallback to `gtk\_file\_chooser\_native\_new` which is used when running in a Flatpak environment.

This patch enabled the native dialog for all environments.

## How to apply and install

1. Add build dependencies: `sudo apt build-dep evolution`
2. Check out this repository
3. Remove any old packages from previous build: `rm *.deb`
4. `cd` into the newly created directory
5. Download source files: `apt source evolution`
6. Switch into source directory and apply each patch (or only the ones you want to):

```
cd evolution-3.46.4 && for n in ../patches/*.patch; do patch -p1 < "$n"; done
```

7. Increment local version number: `EMAIL=nobody@email.com dch -n ''`
8. Start building the packages: `debuild -b -uc -us`
9. Install the required packages: `sudo dpkg -i evolution_*_amd64.deb evolution-common_*_all.deb evolution-plugins_*_amd64.deb libevolution_*_amd64.deb`


