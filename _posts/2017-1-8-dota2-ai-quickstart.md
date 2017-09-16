---
layout: post
title: Dota 2 AI Quick Start
---

Dota 2 7.00 ([The New Journey](http://www.dota2.com/700/other/)) introduced Bots API, which is quite good news for people like me. I am a fan of the game but I never manage to have enough skills or APM to compete with online players. So being able to write my own bots must be a lot of fun. However the official document is very bare bone and unfriendly for people who are new to Dota 2 modding. So here is a simple quick start for people who want to start writing Dota 2 bots.

## Your Working Directory

Find your Dota 2 installation directory, let's call it `$DOTA`. Go to `$DOTA\dota 2 beta\game\dota\scripts\vscripts` and you will find a `bots_example` directory. Make a copy of that, rename it to `bots` — all bots code resides inside that directory. If you start a practice match with bots, it should read bot scripts from that location.

Now you can take a look at [official reference](https://developer.valvesoftware.com/wiki/Dota_Bot_Scripting) (minus the reference, just read the introduction). Here is TL;DR:

1. Bot scripts can be either *override mode* or *completely takeover*. The former only override certain mode, which is a good starting point for new developers.
2. Bot scripts are recognized by file name. So just create Lua scripts with corresponding names and Dota 2 will try to read from them.
3. Lua is used as the scripting language. If you are not familiar with it, I happens to have written a pragmatic quickstart guide, which is intended for experienced developers: [Part I](http://ruoyusun.com/2013/03/23/pragmatic-lua-basics-in-30-mins.html), [Part II](http://ruoyusun.com/2013/03/30/pragmatic-lua-error-handling-oop-closure-and-coroutines.html)
4. Lots of in game data (like heroes, abilities, items) is located at `$DOTA\dota 2 beta\game\dota\scripts\npc`, which is a good reference. **Game item names** can be found [here](https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting/Built-In_Item_Names), which is not mentioned on the wiki.

## Start a Bot vs. Bot Match

1. Enable Dota 2 console by adding `-console` launching options.

2. For bot vs. bot match to work, the example script need to be modified. Open  `hero_selection.lua` and you can see the script tries to select `0-4` for radiant and `5-9` for dire. However, this is only true if you are in the game. For bot vs. bot match, `0-1` is reserved (maybe for coaches?) so radiant is `2-6` and dire is `7-11`.

3. Create a lobby (Play Dota -> Create Lobby) and edit lobby settings. Choose *Local Dev Script* for Radiant and choose any difficulty **except none**. Also tick *Enable Cheats*.

   ![Lobby Settings](http://image.prntscr.com/image/9d92bc777a52417e9300edf1d8682409.png)

4. Make yourself either a coach or unassigned player and start the match.

## Speed Up Development

1. **Show console**: after you have added `-console` launch options, you can toggle console with a hotkey (default is `\`). You can find all the debug info in console, including your print() statement in Lua script, which is very useful.

2. **Reload code**: after you change your bot scripts, it will **not** automatically reload if you have an on-going game. You can use `dota_bot_reload_scripts` command in console to reload your bot scripts. The is the single most important trick I've learned and I am surprised that it is not mentioned in the wiki. 

   *Also please note there is a bug in recent Dota 2 (as of writing) that will crash the game if reload bot scripts in bot vs. bot lobby game. So for now you can start a practice match instead.*

3. **Speed up game**: if you are testing your bot strategies, you might want to speed up the game. Use `host_timescale 4.0` to make the game run at 4x speed. You can change the `4` to anything you prefer. You have to enable cheats to achieve this (use `sv_cheats 1`).

## More Resources

1. [Bot Scripting Wiki](https://developer.valvesoftware.com/wiki/Dota_Bot_Scripting)
2. [Script API Wiki](https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting/API)
3. [Moddota Wiki](http://docs.moddota.com/)
4. [/r/dota2AI](https://www.reddit.com/r/dota2AI/)
5. [Bot Scripting forum](http://dev.dota2.com/forumdisplay.php?f=497)
6. [furiouspuppy's bot scripts](https://github.com/furiouspuppy/Dota2_Bots)
7. [My own bot scripts (very premature)](https://github.com/insraq/dota2bots)
8. **Finally and most importantly, happy hacking!**










