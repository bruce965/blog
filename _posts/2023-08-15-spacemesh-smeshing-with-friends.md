---
title: "Spacemesh: Smeshing with Friends"
date: 2023-08-15 09:29:53 +0200
categories: [Cryptos, Spacemesh]
tags: [cryptos, Spacemesh]
img_path: /assets/img/posts/2023/
---

This guide will help you generate Spacemesh PoS data from a friend's computer
and transfer it to your own personal computer afterwards.

To generate PoS data in a reasonable time you will need a gaming GPU, but you
will only need to generate the PoS data once. Once you have it, you will keep
generating rewards forever as long as you keep smeshing, no GPU needed.

If you don't personally own a gaming GPU and you want to borrow it from a
friend, this guide is for you.


## Smeshing

The first step is to install SMAPP and configure it on your friend's computer.

You can follow my
["Spacemesh: Smeshing with SMAPP"]({% post_url 2023-08-06-spacemesh-smapp-smeshing %})
guide, I would recommend taking a USB hard drive with you, and setting the PoS
directory to a new empty directory on that drive.

![SMAPP: proof of space directory](smesh-6-smeshing-2.png){: .shadow w='1316' h='763' }
_Select the directory where you want to store your data, and click "Next"._

After PoS data has been generated (when the dot next to "Smeshing" turns green),
you can quit SMAPP and proceed to the next step.

![SMAPP: generating data](smesh-7-smeshing-poet.png){: .shadow w='1316' h='763' }
_Initialization completed._


## Exporting Settings

Press `WinKey + R` to open the "Run" prompt.

> The `WinKey` is the one with the Windows logo, usually between `Ctrl` and
> `Alt` near the bottom left corner of the keyboard.
{: .prompt-tip }

In the "Run" prompt, type `%AppData%/Spacemesh` and click "OK".

![SMAPP: run](smesh-8-run.png){: .shadow w='399' h='206' }
_Type "%AppData/Spacemesh%" in the "Run" prompt._

This will open the directory where all SMAPP settings are stored.

![SMAPP settings directory](smesh-9-appdata-light.png){: .light .shadow w='884' h='550' }
![SMAPP settings directory](smesh-9-appdata-dark.png){: .dark .shadow w='884' h='550' }
_This is the SMAPP settings directory._

You will have to transfer this directory to your computer, you can copy it
somewhere on the USB hard drive that you brought to your friend.

> Make sure to delete this folder from your friend's computer. If your friend
> accidentally starts SMAPP with your settings while you are also running it,
> your node might be [permanently banned](https://spacemesh.io/blog/community-clarification-equivocation/).
{: .prompt-warning }

You might also want to uninstall SMAPP from your friend's computer if they are
not interested to become smeshers.


## Importing Settings

Once you are home, connect the USB hard drive and press `WinKey + R` again.

In the "Run" prompt, type `%AppData%` and click "OK".

> Make sure that there is no _Spacemesh_ folder here. If there is any, quit
> SMAPP and rename it from _Spacemesh_ to _Spacemesh\_OLD_.
{: .prompt-warning }

Copy the _Spacemesh_ folder from the USB hard drive to here.

Now look for a file called _node-config.7c8cef2b.json_, open it with a text
editor like Notepad, and make sure that the data directory is correct.

![Node configuration](smesh-10-config.png){: .shadow w='759' h='406' }
_This is the node configuration._

> This file is in JSON format, and the `\` character has a special meaning in
> JSON, therefore you will have to double it when setting the path.
>
> As an example, `C:\MySpacemesh\PoST` becomes `C:\\MySpacemesh\\PoST`.
{: .prompt-warning }

That's all. You may now start SMAPP and start smeshing! 😁


## What now?

If you want to know when you will start seeing your first rewards, you can read
my article ["Spacemesh: Cryptocurrency for the People"]({% post_url 2023-08-06-spacemesh-cryptocurrency-for-the-people %}).

If you have questions, feel free to
[join Spacemesh on Discord](https://discord.com/invite/yVhQ7rC), you will find
smeshers excited to help each other out.

Happy smeshing! 🎉

> P.S.: I am one of the many proud smeshers, here's my wallet's address if you
> want to tip me.
> ```sm1qqqqqqzd25drytuyjy3fmyml9td4aja3vp5rztgcrcucu```
