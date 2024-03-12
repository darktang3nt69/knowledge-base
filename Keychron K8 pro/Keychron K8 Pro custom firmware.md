## Setup

There are a few things we need to do in preparation. Which is basically clone the Keychron fork of QMK, and install the QMK CLI.

### Keychron Repository

-  Clone the repo in a directory of your choosing, and change directories to the newly-cloned repo

```
git clone https://github.com/Keychron/qmk_firmware.git && cd qmk_firmware
```

- In the next section, QMK will call out not having the official firmware as an upstream, so let's satisfy that request:

```
git remote add upstream https://github.com/qmk/qmk_firmware.git
```


- Likewise, qmk setup will tell you the submodules aren't initialized, so let's go ahead and do that too while we are here

```
make git-submodules
```

- We are currently in the "master" branch of the repo, but the K8 pro stuff won't work in this branch (yet?). There is a branch called 'k8_pro', but that one doesn't get it all either. The 'bluetooth_playground' is the one we need. So let's switch to that branch

```
git switch bluetooth_playground
```

- Since, I already have a repo with custom layout. use the following repo. Make sure to sync upstream.
```
git clone https://github.com/darktang3nt69/qmk_firmware.git && cd qmk_firmware
git switch bluetooth_playground
```

### QMK CLI

- Most of the information on how to get this set up is available on [QMK's setup guide](https://qmk.github.io/qmk_mkdocs/master/en/tutorial_getting_started/). Basically we need to install the QMK CLI with pip

```
python3 -m pip install --user qmk
```

- And then running:

```
qmk setup
```

- If you followed along above, you probably don't have anything to fix; But if something does come up, you'll probably want to answer 'y' to anything that comes up and let it fix whatever it finds. At this point you are ready to actually compile a firmware.

### Optional setup

- Since you probably only have one keyboard, we can tell qmk to just always use that keyboard with `qmk config user.keyboard=keychron/k8_pro/ansi/rgb` if you don't specify that, you will need to pass `-kb keychron/k8_pro/ansi/rgb` to every qmk command.

## Testing a build

- There are two keymaps defined, one called _default_ and one called _via_. The only difference between them is a rules.mk in the via one that enables via support. So if you use default, via won't work. So with that, lets try to build the via keymap.

```
qmk compile -km via
```

- Now we can create a custom keymap. The qmk cli has a qmk new-keymap, which will prompt you for a new keymap name, but you can also specify with -km mynewmap. That will essentially just create a new keymap based on default. Since we wanted to maintain via compatibility, we could just copy the rules.mk from via into our new keymap directory, but since it is just copying anyway, I created a new keymap with:

```
cp -R keyboards/keychron/k8_pro/ansi/rgb/keymaps/via keyboards/keychron/k8_pro/ansi/rgb/keymaps/mynewmap
```

- At this point you could try to compile your new one (which should be the same as the via one that we tested above) with

```
qmk compile -km mynewmap
```

- Now with that change, we are going to compile for real this time:

```
qmk compile -km mynewmap
```

- Assuming everything is good, we can move on to flashing.

## Flashing

- The previous step should have created a keychron_k8_pro_ansi_rgb_mynewmap.bin file. We are going to flash the keyboard with that file. We are again going to use the qmk CLI to do that. Run:

```
qmk flash keychron_k8_pro_ansi_rgb_mynewmap.bin
```

- You should see:

```
Flashing binary firmware...
Please reset your keyboard into bootloader mode now!
Press Ctrl-C to exit.
```

- Ok, so let's do what it tells us. If you follow the instructions on Keychron's site, in order to do this you have to do the following:

```
1. move the keyboard switch to the 'cable'
2. Pop off the spacebar keycap
3. press and hold the reset button
4. plug in the USB cable
5. release the reset button
```

- This works, and I think the point is that this works no matter what. I don't particularly want to pop off my keycap every time I want to flash a firmware, and it turns out there is a simpler way. The USB cable can already be plugged in for this method (probably could be for the previous one too, but their documentation specifically states to have it unplugged)

```
1. move the keyboard switch to off
2. plug in the USB cable (if not already connected)
3. hold the escape key (upper left-hand button)
4. move the keyboard switch to cable
5. release the escape key.
```

1. The process is basically the same, with the major difference being you don't have to pop off any keycaps to do it.

2. If all goes well, you should see the qmk flash command begin to wipe the keyboard memory and apply the new firmware. Then we should be ready to go. Except, if you hit your new capslock-thats-supposed-to-be-an-escape, you'll still see caps lock behavior. WTF?

Well it turns out this is a via-compatibility thing. This is specifically covered in [the FAQ](https://docs.qmk.fm/#/faq_keymap?id=my-keymap-doesnt-update-when-i-flash-it). The solution is to basically do the thing we did above to put it in bootloader mode:Hold the Bootmagic Lite key (usually top left/Escape) while plugging the board in, which will also place the board into bootloader mode; then unplug and replug the board.

Since the K8 pro has a physical switch, you can do the same thing with switching it from off->cable instead of the plugging/unplugging.