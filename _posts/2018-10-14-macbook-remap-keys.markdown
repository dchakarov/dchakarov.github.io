---
title: 'MacBook Keyboard Setup: The Mysterious § Key'
date: 2018-10-14 00:00:00 Z
layout: post
---

<span class="dropcap">W</span>hen you start a new job in an office, you usually get a new laptop. If your job is an iOS developer, you probably get a MacBook Pro. The first few days are usually dedicated to setting up your environment, big part of which are the keyboard layout and shortcuts. My first job is to install the Bulgarian keyboard layout and to remap Cmd-Space to switch between English and Bulgarian. I realise this is too specific to me and as such probably has no value to you. A thing a bit more useful for you - the second thing I do is swap the two shortcuts for screenshots. That means Shift-Cmd-4 to save the screenshot of the selected area into the clipboard and Ctrl-Shift-Cmd-4 to save to a file. I realised I was pasting screenshots into chats way more than I was needing them saved for later.

The third thing I usually do is to install an app like [Karabiner](https://github.com/tekezo/Karabiner-Elements) so I can swap and remap keys. The most important keys for me to swap are the **§** and the **\`** key. That's because I tend to switch between multiple windows in the same app often (mainly in Xcode and SourceTree) and I am used to doing that with Cmd + the key above Tab. On the MacBook keyboard that key is **§** and the shortcut doesn't work without some outside help. Some people swap other keys too, like **Caps Lock** and **Esc**, but I am not gonna talk about this today.

Karabiner is a great app, but at my new company the laptops are a bit more restricted and I was unable to install it. I googled for a solution and - unsurprisingly - no-one else had had the same issue. Either people don't use that shortcut or they are content with the new location of the **\`** key. Well I was not. Luckily, people were not content with the Esc key and there were a lot of howtos describing how to map **Caps Lock** to be your new **Esc** key. Following a couple of [answers on Stack Overflow](https://stackoverflow.com/questions/127591/using-caps-lock-as-esc-in-mac-os-x/46460200#46460200), I ended up on [Remapping Keys in macOS 10.12 Sierra](https://developer.apple.com/library/archive/technotes/tn2450/_index.html#//apple_ref/doc/uid/DTS40017618-CH1-KEY_TABLE_USAGES) on Apple's website. As the link might die in the future, I'll try to paraphrase here. To remap i.e. the **Caps Lock** key to **Esc**[^1] you can use a simple command (introduced in macOS Sierra):

``` bash
$ hidutil property --set '{"UserKeyMapping":[{"HIDKeyboardModifierMappingSrc":0x700000039,"HIDKeyboardModifierMappingDst":0x700000029}]}'
```

- In this command HIDKeyboardModifierMappingSrc is the key you want to remap and the HIDKeyboardModifierMappingDst is the new value for that key.
- The values in this example are from the lookup table on Apple's website page linked above.
- This do not swap the keys, it just remaps the first one to the second one.
- Executing multiple such commands won't remap all the keys from all the commands. We are setting a property, so every command will overwrite the previous one. If you want to remap multiple keys, include them into a single command, e.g. `$ hidutil property --set '{"UserKeyMapping":[{"HIDKeyboardModifierMappingSrc":0x700000039,"HIDKeyboardModifierMappingDst":0x700000029},{"HIDKeyboardModifierMappingSrc":0x700000029,"HIDKeyboardModifierMappingDst":0x700000039}]}'`
- If you want to delete your mapping, execute the command with an empty array like so: `$ hidutil property --set '{"UserKeyMapping":[]}'` or restart your Mac.

<img src="{{ '/assets/img/keys-lookup-table.png' | prepend: site.baseurl }}" alt="">

There is one problem with the lookup table - the **§** key is not in it!

I started testing, trying to get to it and, a few minutes later, I found it - *0x700000064*. I executed the command and - ta-da! - the **\`** key was back where it belonged!

``` bash
$ hidutil property --set '{"UserKeyMapping":[{"HIDKeyboardModifierMappingSrc":0x700000064,"HIDKeyboardModifierMappingDst":0x700000035}]}'
```

But wait, we're not done yet. We need that swap to be active forever without you having to run it again every time you restart your machine. My first instinct was to put the command into a .sh file and add it to the Login items in the System Preferences of the Mac. I tried it but for some reason it didn't start the next time I restarted. So I dug deeper and found out I had to create an Automator script and compile it to a binary (app) which I can then safely add to the Login items. A few minutes later I had the final solution :)

The app and the script are [available here](https://github.com/dchakarov/restore-tilde).

[^1]: Since macOS 10.12.1 you can remap **Caps Lock** to **Esc** without using a specialised command. (via [9to5Mac](https://9to5mac.com/2016/10/25/remap-escape-key-action-macbook-pro-macos-sierra-10-12-1-modifier-keys/) et al.)