---
title: config_version 6
---

# Changes in MPF config version 6

MPF uses version numbers in its YAML config files which ensure that MPF is loading
files that are compatible with the version of MPF that's running. [More info on config versions](config_version.md)

MPF 0.57 moves from config version 5 to config version 6 (and from show version 5 to 6 as well). This page documents the changes.

## Why config version 6?

Back when MPF was new, we added some "hacks" to the YAML processor to make it easier to write config files. At the time we thought this would make the YAML files easier to use for non-programmers. While that was somewhat true, it turns out that the "hacks" we added made it harder to maintain MPF and to add new features. (These hacks are part of the reason it took us so long to support Python versions newer than 3.9.)

Those hacks are being removed in MPF 0.57, which means you'll need to update your config files so they use "pure" YAML and don't include any of our older hacks. Luckily this is a pretty straightforward process, mostly doing some "find and replace". (Details below) As part of that, you'll also need to update your config files to config version 6.

## Specific changes you need to make

If you want to know what specific hacks MPF used prior to config version 6, see the
"What were the hacks?" section below. But if you just care about updating your YAML files,
here's what you need to do:

1. Find and replace `#config_version=5` with `#config_version=6`.
2. Find and replace `#show_version=5` with `#show_version=6`.
3. Find and replace `: +` with `: "+`. You'll need to also add the quote to the end of the line. Or if you only have a few different values, you can find and replace the entire line, like `time: +1` with `time: "+1"`, `time: +2` with `time: "+2"`, etc (however, do NOT do this around any `time: 0` entries in shows).
4. Search for any value that starts with a leading zero, like `: 0` and then see if it only has digits after the zero. If so, add quotes around the value. e.g. `: "000066"`. If this is a multi-part value, put quotes around the whole thing: `number: 0804-1` becomes `number: "0804-1"`. This also applies for key names: `0804:` becomes `"0804":`.
5. Any color values that are only numbers will need quotes around them. e.g. `color: 330000` becomes `color: "330000"`.
6. Search your YAML files for `!!omap` and remove and update those. (See below.)
7. Update the format of your high score data files. (See below.)
8. You might have a bit of cleanup for some random other things which are now invalid YAML (as outlined below), but the easiest way to do that is just to run your game and then hunt down any last remaining errors as they come up.

That's it! Not too bad overall. We updated several configs for complete and mature machines, and
each machine's entire bundle of configs and shows only took a few minutes. It's really pretty quick.

## What were the hacks?

Here are hacks that MPF used prior to config version 6:

**Values beginning with "+" are strings**

   The YAML spec views values that begin with a plus sign as numbers. So a line like `time: +1`
   is the same as `time: 1`. But for MPF, these mean different things. (e.g. in shows, a time of
   `+1` means "one second after the previous step", while `1` means "one second after the show start".)

   The fix for this is to add quotes around the values that start with plus. e.g. `time: "+1"` instead of `time: +1`.

**Values beginning with a leading "0" are strings**

   The YAML spec will process values that are only digits with leading zeros as numbers.
   So in pure YAML, `color: 000066` would be processed as `color: 66` which is wrong.

   The fix for this is to add quotes around the values that are only numbers and start with a leading zero. e.g. `color: "000066"` instead of `color: 000066`.

**Values with only digits and "e" are strings**

   The YAML spec will process a value like ``123e45`` as "123 exponent 45". Since those could
   be hex color codes, MPF's YAML interface processes values that are all digits with a single
   "e" character as strings.

**!!omap is no longer needed**

   MPF used to use the `!!omap` YAML type to ensure that the order of items in a list was
   preserved. This is no longer needed with the current versions of Python, so you can remove
   the `!!omap` from your YAML files.

   If your keys in the section had leading dashes, you need to remove those as well.

   For example, this:

   ``` yaml title="Old way"
   position_switches:  !!omap
   - up: s_position_up
   - down: s_position_down
   ```

   Becomes this:

   ``` yaml title="New way in config version 6"
   position_switches:
     up: s_position_up
     down: s_position_down
   ```

   Notice the dashes that were in front of the keys are gone.

   If the values in the omap section were just keys with no colons and no values, then you need to
   remove the !!omap but keep the dashes, like this:

   ``` yaml title="Old way"
   categories:  !!omap
    score:
      - GRAND CHAMPION
      - HIGH SCORE 1
      - HIGH SCORE 2
   ```

   Becomes this:

   ``` yaml title="New way in config version 6"
   categories:
    score:
      - GRAND CHAMPION
      - HIGH SCORE 1
      - HIGH SCORE 2
   ```

## How to update your high score data file

If you use MPF's High Score mode, your high scores (and other achievements) are stored in a YAML file. The format of that file has changed in config version 6. Here's how to update it:

1. Find your high scores file. It will most likely be in your machine folder, in the data folder, and it will be named `high_scores.yaml`.

2. Make some changes. Here's what an old high score file will look like:

``` yaml title="Old high score file"

bonus_rupees:
- !!python/tuple
  - WIZ
  - 146
- !!python/tuple
  - AAA
  - 126
score:
- !!python/tuple
  - WIZ
  - 563550
- !!python/tuple
  - WIZ
  - 537200
- !!python/tuple
  - WIZ
  - 510630
- !!python/tuple
  - WIZ
  - 482705
- !!python/tuple
  - AAA
  - 458435
- !!python/tuple
  - WIZ
  - 449960

```

3. Delete the `!!python/tuple` from each line. (You can do this with a find and replace.) However you want to keep the dash, so your new file will look like:)

``` yaml title="New high score file for MPF 0.57"

bonus_rupees:
- - WIZ
  - 30
- - WIZ
  - 25
score:
- - WIZ
  - 100000
- - WIZ
  - 95000
- - WIZ
  - 90000
- - AAA
  - 86490
- - WIZ
  - 85000
- - WIZ
  - 80000
```
