---
layout: post
title:  "Releasing a PC demo featuring new music by a legendary Amiga composer"
published: true
tags: Amiga, PC, demoscene, TDK
---
In 1991 there was a popular [Amiga intro](https://www.youtube.com/watch?v=a9TB358j2qM) that was used by a well known group at the time. That demo featured a smash hit song called *maintheme2*, composed by the British demoscener TDK. For many people, this is a nostalgic memory of Amiga intros from the 90's. An intro could be a reason itself for firing up a disk with the demo on it, just to enjoy the show. 

Anyway, some time ago I was experimenting with some CSS3 animations with a plan to release something at a Commodore 64 demoparty taking place some months later. Even though that was a pure 8 bit event, I thought I could show something out of competition, just for the heck of it. I had a couple of effects, but no music. 

Then I came to think about that old song in the Amiga intro. Why not look up TDK and ask him if he has something unreleased in his treasure chest?

And that was what I did. And TDK told me he would look out for something, and try to get something sorted. Most of his old music had already been released on a couple of music disks though. 

But then, like a bolt from the blue, TDK told me he had been working on a remix of maintheme2, however not finished yet. He had the backing track as he [played it](https://youtu.be/q_q1lzlGlLw?t=4m1s) on the violin at Revision a few years ago. There was still quite a bit of work to do. He told me he had left it alone for a while, but that it would be released on a download album some day. And then, guess what, TDK generously told me he'd try to finish it so I could use it in some demo. Hey, that would be a blast, wouldn't it!

The music format in a traditional Amiga demo from the 90's is [MOD](https://en.wikipedia.org/wiki/MOD_(file_format)) or [XM](https://en.wikipedia.org/wiki/XM_(file_format)), not MP3, Ogg or any other modern audio coding formats. This new song was an MP3, so it would fit a browser demo, it certainly would. But I changed my mind and decided to code something for Windows instead. But I was in a hurry! The Commodore 64 demoparty was coming up in a week. And the song wasn't finished. 

TDK did some work on the mix, got the melody in, and then finally he had it done. Meantime I rushed with the PC demo, putting together the same effects I did with CSS3 now in C# instead. Maybe I could add some additional effects as well.

MonoGame
========
[MonoGame](http://www.monogame.net/) is a C# framework implementing the Microsoft XNA 4 API, supporting not only Windows but also OS X, Linux, iOS, Android and other platforms. This is a fun environment, making it easy to get started. If you have some effect ideas you just want to try out, MonoGame can be a good choice just to get something on the screen.

I ended up with a couple of classic demo effects:

Rotozoomer
==========
A *rotozoomer* is a screen rotating and zooming at the same time, an effect invented by Chaos from the German demo group [Sanity](https://en.wikipedia.org/wiki/Sanity_(demogroup)) in his demo [World of Commodore](https://www.youtube.com/watch?v=u43uH-kQpzk) in 1992. You can try it out yourself starting with some source code from [GitHub](https://github.com/FransBouma/CSRotoZoomer), or you could do it the simple way just by updating the `rotation` and `scale` arguments in `Draw`:
{% highlight csharp %}
internal void Draw(SpriteBatch spriteBatch, int alphaValue)
{
    spriteBatch.Draw(texture, screenPosition, null, 
                    Color.White * ((float)alphaValue / 255), 
                    base.RotationAngle, origin, 
                    base.Scale, 
                    SpriteEffects.None, 0f);
}

internal void Update(GameTime gameTime)
{
    base.ZoomInOut(gameTime);
    base.Rotate(gameTime);
}
{% endhighlight %}

And in the base class:
{% highlight csharp %}
public virtual void ZoomInOut(GameTime gameTime)
{
    scaleDelay -= gameTime.ElapsedGameTime.TotalSeconds;
    if (scaleDelay <= 0)
    {
        scaleDelay = scaleDelayOriginal;
        Scale += ScaleIncrement;
        if (Scale >= ZoomUpperBound || 
            Scale <= ZoomLowerBound)
        {
            ScaleIncrement *= -1;
        }
    }
}

public virtual void Rotate(GameTime gameTime)
{
    float elapsed = (float)gameTime.ElapsedGameTime.TotalSeconds;
    RotationAngle += elapsed;
    float circle = MathHelper.Pi * 2;
    RotationAngle = RotationAngle % circle;
}
{% endhighlight %}


Rotating boxes
==============
Some boxes, with rotation, scaling and coloring. Nothing complex.

Plasma
======
This is very much the plasma I made with [JavaScript](http://kristofferjalen.github.io/2016/05/19/an-amiga-style-demo-remade-with-css3-and-javascript/) a couple of months ago, based on code from [Lode's computer graphics tutorial](http://lodev.org/cgtutor/plasma.html).

Generate a palette of RGB colors. Make sure you generate a palette without discontinuities and a palette that can be wrapped around (tileable).
{% highlight csharp %}
palette = new int[256][];
for (int x = 0; x < 256; x++)
{
    palette[x] = new int[] { 
        (int)Math.Floor(128 + 
                    128 * Math.Sin(Math.PI * x / 32)),
        (int)Math.Floor(128 + 
                    128 * Math.Sin(Math.PI * x / 64)),
        (int)Math.Floor(128 + 
                    128 * Math.Sin(Math.PI * x / 128))
    };
}
{% endhighlight %}

Then generate the plasma buffer that contains the result of the sine function for every pixel:
{% highlight csharp %}
plasma = new int[texture.Width * texture.Height][];
float zoom = .1f;
for (int y = 0; y < texture.Height; y++)
{
    int[] p = new int[texture.Width];
    for (int x = 0; x < texture.Width; x++)
    {
        p[x] = (int)Math.Floor(Math.Floor(128.0 
            + 128.0 * Math.Sin(zoom * x / 16.0)
            + 128.0 + 128.0 * Math.Sin(zoom * y / 8.0)
            + 128.0 + 128.0 * Math.Sin(zoom * (x + y) / 16.0)
            + 128.0 + 128.0 * Math.Sin(
                Math.Sqrt((double)zoom * zoom 
                            * (x * x + y * y) / 8.0))) / 4);
    }
    plasma[y] = p;
}
{% endhighlight %}

And then in `Update` you just redraw every pixel with a shifted palette color:
{% highlight csharp %}
paletteShift += 2;
for (int y = 0; y < 741; y++)
{
    for (int x = 0; x < 1200; x++)
    {
        if (x >= 150 && x <= 1021 && y >= 155 && y <= 587)
            continue;

        int[] c = palette[(plasma[y][x] + paletteShift) 
                            % 256];
        colors[x + y * 1200] = new Color(c[0], c[1], c[2]);
    }
}
texture.SetData(colors);
{% endhighlight %}


Interference circles
====================
Move around some circles and when circles are crossed, colors are mixed. One famous demo with this effect is [State of the Art](https://www.youtube.com/watch?v=wCc5ZHqwdXY&t=40s) by Spaceballs from 1992. 

With inspiration from an [article](http://www.csharpskolan.se/article/monogame-old-school-demo) about the effect, this was a quite nice part.    

The result
==========
[Download](http://www.pouet.net/prod.php?which=67733) the demo and run it your own PC, or see a screen capture of the demo right here:

<iframe width="720" height="540" src="https://www.youtube.com/embed/uKAjzUI0ZIs" frameborder="0" allowfullscreen></iframe>

What did you think? I'm happy to hear your comments!