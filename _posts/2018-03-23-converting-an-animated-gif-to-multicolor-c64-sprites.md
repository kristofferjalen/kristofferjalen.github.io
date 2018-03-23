---
layout: post
title:  "Converting an animated GIF to multicolor C64 sprites"
published: true
tags: .NET, C64, demoscene
comments: true
---
Recently I stumbled upon a nice little [GitHub project](https://github.com/jeff-1amstudios/gif-to-c64-sprites) that converts animated GIFs to Commodore 64 sprites. That tool converts to **high resolution** sprites, so I wrote a [.NET tool](https://github.com/kristofferjalen/gif-to-c64-sprites) that converts to **multicolor** sprites.

As described by [C64-Wiki](https://www.c64-wiki.com/wiki/Sprite), in multicolor sprites the bits are grouped in pairs, and since each such multicolor pixel is defined by two bits of data rather than one, each pixel can do one of four things:

- Pixels with a bit pair of `00` appear transparent, like "0" bits do in high resolution mode
- Pixels with a bit pair of `01` will have the color specified in address $d025
- Pixels with a bit pair of `11` will have the color specified in address $d026
- Pixels with a bit pair of `10` will have the color specified assigned to the sprite in question in the range $d027–d02e

Thus each color corresponds to a specific delegate that takes the bit position of the pixel and returns a value to add to the byte that the pixel belongs to:

{% highlight csharp %}
public static IDictionary<C64Colors, Func<int, int>> ToByteAdds(this ColorsString colorsString) =>
    new Dictionary<C64Colors, Func<int, int>>
    {
        {colorsString.Value[0].ToC64Color(), x => 0},
        {colorsString.Value[1].ToC64Color(), x => (int)Math.Pow(2, 6 - x)},
        {colorsString.Value[2].ToC64Color(), x => (int)Math.Pow(2, 7 - x) + (int)Math.Pow(2, 6 - x)},
        {colorsString.Value[3].ToC64Color(), x => (int)Math.Pow(2, 7 - x)}
    };

In this method, `colorsString` is a four characters string of hex numbers defining the color in each of the registers $d020, $d025, $d026, $d027.

## Party Parrots

Party parrots in Slack are animated drawings of [Sirocco the kakapo parrot](http://knowyourmeme.com/memes/party-parrot). I converted some of the lovely parrots on [http://cultofthepartyparrot.com/](http://cultofthepartyparrot.com/) and let these little fellas slide in to the screen.

The result was pretty funny:

<iframe width="560" height="315" src="https://www.youtube.com/embed/q8wslJgwOWo?rel=0&amp;showinfo=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

The repo contains a simple [C64 demo](https://github.com/kristofferjalen/gif-to-c64-sprites/tree/master/c64-sample-app) so you can go try it out yourself!