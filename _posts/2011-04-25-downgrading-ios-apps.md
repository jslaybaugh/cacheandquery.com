---
layout: post
title: Downgrading iOS Apps
tags:
- apps
- iOS
- iTunes
- Technology
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  dsq_thread_id: '288503188'
---

Last week I upgraded several apps on my iPhone and usually that process is smooth and results in awesome new features/bug fixes for all the updated apps. This time, unfortunately, the opposite was the case. [Zynga recently bought the "with Friends" franchise of apps from Newtoy, Inc][1], and their update of Chess With Friends made the app practically unusable. The new graphics were pretty ugly, in my opinion, and the game play was painfully slow. So, it wasn't hard to ultimately decide to roll back the install until Zynga gets all this worked out. However, I'd never actually had to roll back an app before. Once I got it figured out, I thought I'd post here to help anyone else out. These instructions are written for a PC, but the process on a Mac would be pretty similar.

The easy way first. If you updated from your device and haven't updated on your PC yet, just delete the app and sync with your PC to get the version from the last device sync. However, if it were always that easy, then there wouldn't be a need for this post. So, now, the hard(er) way...

This is assuming you've updated the app both on your PC and on your device and therefore cannot just simply delete and re-sync. If that is the case, first start by deleting the app on your device (don't worry, if this doesn't work, you can always re-download the latest version from the iTunes Store and you'll be no worse for wear... even if its a purchased app).

![Deleting an app](/img/posts/iphone_shots.png)

Then go into your Mobile Applications folder on your PC (usually C:Users[your user name]My MusiciTunesiTunes MediaMobile Applications but your folder may be different depending upon your versions of Windows and iTunes). If you can't find it, you can just search for *.ipa files and go from there. In the screenshot below, you can see that we've found the old version of the app we want (the new version is 4.01 and the old version is 3.07). Copy that file to your desktop or somewhere to make sure you have a good copy. If you can't find the file in that folder, do a search in your Recycle Bin for *.ipa files. iTunes is nice enough to put old files there as it updates, so usually you can find it there if you can't find it in your Mobile Applications folder.

![Finding the .ipa file](/img/posts/pc_find_ipa.png)

Once you have it on your desktop, drag it to your iTunes Library. It will ask if you want to overwrite the current version. Of course you do! From there, now the right version is in your library, so you can just sync with your device and you should be good to go! (Don't forget to re-select the app in iTunes to sync to your device).

![Drag to iTunes](/img/posts/itunes_drag.png)

![Silly prompt](/img/posts/itunes_prompt.png)

![Make sure and sync the app](/img/posts/itunes_sync.png)

Now, the only thing to be aware of is that when you download future updates on your device or computer, you'll have to avoid downloading the update for the app you downgraded by not choosing "Update All" and instead updating all the other apps one-at-a-time. It becomes kind of a pain, and you'll always have that annoying red "1" on your device's App Store icon letting you know there's an update, but hopefully the company in charge of the bummed app (I'm looking at you, Zynga) will get it straight soon and you won't have to put up with that for long.

Have fun!

P.S. If, by chance, you are reading this and you need the old version of Chess With Friends because you can't find it on your computer anywhere, [here][2] it is in a zip file.

 [1]: http://www.tuaw.com/2010/12/02/zynga-buys-newtoy-studio-rebranded-as-zynga-with-friends/
 [2]: /files/chess_3.07_1.ipa.zip  

