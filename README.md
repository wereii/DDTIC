# DDTIC

This repository contains a short tutorial on how to completely disable
Discord changing it's icon in tray on Linux machine.

## Why

On my system it actually does not work at all, I've never seen 
the microphone icon or anything else **and** something to do with this
eventually completely freezes the UI of Discord when I am calling for 
longer time (1+ hours).

This is on `Manjaro Linux with AwesomeWM, 16GB, i7-7700HQ, nVidia 1060M`.

## How To Disable Tray Icon Change And Fix Discord Freezing During Longer Calls

*Please read the Disclaimer at the end first before doing anything*

Also if you are stuck, or don't understand what I mean in any step, stop now
and create an issue here and I will try to help.

There will be one program needed for this called `asar`
you should be able to find it in your package manager, if not you will have to
install it from here https://github.com/electron/asar.

- Completely stop discord if it's running
- Navigate to `~/.config/discord/0.0.X` where X is your current discord version
  you are looking for highest number here
- In that directory navigate to `modules/discord_desktop_core`. The current path
  should then look like this: `~/.config/discord/0.0.X/modules/discord_desktop_core`
- In this directory there should be a few files, mainly `core.asar` and `index.js`
- Run `asar extract core.asar core/`, this will unpack the `core.asar` 
  into directory named `core`
- Now comes the hardest part, editing the code:
    - Open this file `core/app/systemTray.js` with your favourite editor
    - Either search for line starting with `function setTrayIcon(icon)` or jump to line 
      164.

      Keep in mind that different Discord versions can and will have different stuff
      on exact line numbers, this was developed with 0.0.16 in mind, if you have different
      version then you should pay extra attention where you are pasting the next code.

    - Insert this if:

      ```js
      if (icon == "DEFAULT") {
        return;
      }
      ```

      after the the line 164 
      **or** if you were searching for function, paste it **under** this piece of code
      (within the function `setTrayIcon`): 

      ```js
      if (icon == null) {
        hide();
        return;
      } else {
        show();
      } // Keep mute/deafen menu items in sync with client, based on icon states

      // >HERE COMES THE IF< (replace this whole line, do not keep the double slash at the start)
      ```

    - For verification, the code after this looks like this:

      ```js
      // ...
      function setTrayIcon(icon) {
        // Keep track of last set icon
        currentIcon = trayIcons[icon]; // If icon is null, hide the tray icon.  Otherwise show
        // These calls also check for tray existence, so minimal cost.

        if (icon == null) {
          hide();
          return;
        } else {
          show();
        } // Keep mute/deafen menu items in sync with client, based on icon states

        if (icon == "DEFAULT") {
          return;
        }

        const muteIndex = contextMenu.indexOf(menuItems[MenuItems.MUTE]);
      // ...
      ```

- One last thing remining, we need to switch a track and point Discord to the new unpacked 
  folder instead of the asar file.
  In `index.js`, you have to change the `require('./core.asar')` to `require('./core')`,
  like this:

  ```js
  module.exports = require("./core");
  // module.exports = require('./core.asar');
  ```

- That's it. You will probably have to do this for every new version of Discord.
  It might event break stuff completely, absolutely no guarantees on my side.



# Disclaimer

Obviously you should never paste a random code some rando on the web told you to paste somewhere.
Not into discord, not anywhere. So, is this safe? It works on my machine :tm: :shrug:.

Though if something indeed failed just make an issue here and I will try to help,
either way I am not taking any responsibility for any damage that might come out of this,
it's as they say, for educational purposes so by following this how-to you acknowledge that
you know what you are doing.

