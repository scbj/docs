# JavaScript (a.k.a. JS)

## Converting Color Spaces in JavaScript

_Published: Jan 10, 2019_ by [John Kantner](https://css-tricks.com/author/jon-kantner/)

Source: [CSS-Tricks](https://css-tricks.com/converting-color-spaces-in-javascript/)

A challenge I faced in building an [image "emojifier"](https://codepen.io/jkantner/pen/OoXvMB) was that I needed to change the color spaces of values obtained using ```getImageData()``` from RGB to HSL. I used arrays of emojis arranged by brightness and saturation, and they were HSL-based for the best matches of average pixel colors with the emojis.

In this article, we’ll study functions that will be useful for converting both opaque and alpha-enabled color values. Modern browsers currently support the color spaces RGB(A), hex, and HSL(A). The functions and notations for these are ```rgb()```, ```rgba()```, ```#rgb/#rrggbb```, ```#rgba/#rrggbbaa```, ```hsl()```, and ```hsla()```. Browsers have always supported built-in names like ```aliceblue``` as well.

![img](/images/threads/color-conversion-featured.webp)

Along the way, we’ll encounter use of some color syntaxes provided by a new [Level 4](https://drafts.csswg.org/css-color/) of the CSS Colors Module. For example, we now have hex with alpha as we mentioned (```#rgba```/```#rrggbbaa```) and RGB and HSL syntaxes no longer require commas (values like ```rgb(255 0 0)``` and ```hsl(240 100% 50%)``` became legal!).

::: danger Hey!
Browser support for CSS Colors Level 4 isn’t universal as of this writing, so don’t expect new color syntaxes to work in Microsoft browsers or Safari if trying them in CSS.
:::

### RGB to Hex

Converting RGB to hex is merely a change of radices. We convert the red, green, and blue values from decimal to hexadecimal using ```toString(16)```. After prepending ```0```s to single digits and under, we can concatenate them and ```#``` to a single ```return``` statement.

```javascript
function RGBToHex(r,g,b) {
  r = r.toString(16);
  g = g.toString(16);
  b = b.toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;

  return "#" + r + g + b;
}
```

#### RGB in String

Alternatively, we can use a single string argument with the red, green and blue separated by commas or spaces (e.g. ```"rgb(255,25,2)"```, ```"rgb(255 25 2)"```). Substring to eliminate ```rgb(```, split what’s left by the ```)```, then split that result’s first item by whichever the separator (```sep```) is. ```r```, ```g```, and ```b``` shall become local variables now. Then we use ```+``` before the split strings to convert them back to numbers before obtaining the hex values.

```javascript
function RGBToHex(rgb) {
  // Choose correct separator
  let sep = rgb.indexOf(",") > -1 ? "," : " ";
  // Turn "rgb(r,g,b)" into [r,g,b]
  rgb = rgb.substr(4).split(")")[0].split(sep);

  let r = (+rgb[0]).toString(16),
      g = (+rgb[1]).toString(16),
      b = (+rgb[2]).toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;

  return "#" + r + g + b;
}
```

In addition, we can allow strings with channel values as percentages by adding the loop after redefining ```rgb```. It'll strip the ```%```s and turn what’s left into values out of 255.

```javascript
function RGBToHex(rgb) {
  let sep = rgb.indexOf(",") > -1 ? "," : " ";
  rgb = rgb.substr(4).split(")")[0].split(sep);

  // Convert %s to 0–255
  for (let R in rgb) {
    let r = rgb[R];
    if (r.indexOf("%") > -1)
      rgb[R] = Math.round(r.substr(0,r.length - 1) / 100 * 255);
      /* Example:
      75% -> 191
      75/100 = 0.75, * 255 = 191.25 -> 191
      */
  }

  ...
}
```

Now we can supply values like either of these:
* ```rgb(255,25,2)```
* ```rgb(255 25 2)```
* ```rgb(50%,30%,10%)```
* ```rgb(50% 30% 10%)```

### RGBA to Hex (#rrggbbaa)

Converting RGBA to hex with the #rgba or #rrggbbaa notation follows virtually the same process as the opaque counterpart. Since the alpha (```a```) is normally a value between 0 and 1, we need to multiply it by 255, round the result, then convert it to hexadecimal.

```javascript
function RGBAToHexA(r,g,b,a) {
  r = r.toString(16);
  g = g.toString(16);
  b = b.toString(16);
  a = Math.round(a * 255).toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;
  if (a.length == 1)
    a = "0" + a;

  return "#" + r + g + b + a;
}
```

To do this with one string (including with percentages), we can follow what we did earlier. Also note the extra step of splicing out a slash. Since CSS Colors Level 4 supports the syntax of ```rgba(r g b / a)```, this is where we allow it. Alpha values can now be percentages! This removes the 0-1-only shackles we used to have. Therefore, the ```for``` loop cycling through ```rgba``` shall include a part to wipe the ```%``` from the alpha without multiplying by 255 (when ```R``` is 3 for alpha). Soon we can use values like ```rgba(255 128 0 / 0.8)``` and ```rgba(100% 21% 100% / 30%)```!

```javascript
function RGBAToHexA(rgba) {
  let sep = rgba.indexOf(",") > -1 ? "," : " ";
  rgba = rgba.substr(5).split(")")[0].split(sep);
                
  // Strip the slash if using space-separated syntax
  if (rgba.indexOf("/") > -1)
    rgba.splice(3,1);

  for (let R in rgba) {
    let r = rgba[R];
    if (r.indexOf("%") > -1) {
      let p = r.substr(0,r.length - 1) / 100;

      if (R < 3) {
        rgba[R] = Math.round(p * 255);
      } else {
        rgba[R] = p;
      }
    }
  }
}
```

Then, where the channels are converted to hex, we adjust ```a``` to use an item of ```rgba[]```.

```javascript
function RGBAToHexA(rgba) {
  ...
    
  let r = (+rgba[0]).toString(16),
      g = (+rgba[1]).toString(16),
      b = (+rgba[2]).toString(16),
      a = Math.round(+rgba[3] * 255).toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;
  if (a.length == 1)
    a = "0" + a;

  return "#" + r + g + b + a;
}
```

Now the function supports the following:

* ```rgba(255,25,2,0.5)```
* ```rgba(255 25 2 / 0.5)```
* ```rgba(50%,30%,10%,0.5)```
* ```rgba(50%,30%,10%,50%)```
* ```rgba(50% 30% 10% / 0.5)```
* ```rgba(50% 30% 10% / 50%)```

### Hex to RGB

We know that the length of hex values must either be 3 or 6 (plus ```#```). In either case, we begin each red (```r```), green (```g```), and blue (```b```) value with ```"0x"``` to convert them to hex. If we provide a 3-digit value, we concatenate the same value twice for each channel. If it’s a 6-digit value, we concatenate the first two for red, next two for green, and last two for blue. To get the values for the final ```rgb()``` string, we prepend the variables with ```+``` to convert them from strings back to numbers, which will yield the decimals we need.

```javascript
function hexToRGB(h) {
  let r = 0, g = 0, b = 0;

  // 3 digits
  if (h.length == 4) {
    r = "0x" + h[1] + h[1];
    g = "0x" + h[2] + h[2];
    b = "0x" + h[3] + h[3];

  // 6 digits
  } else if (h.length == 7) {
    r = "0x" + h[1] + h[2];
    g = "0x" + h[3] + h[4];
    b = "0x" + h[5] + h[6];
  }
  
  return "rgb("+ +r + "," + +g + "," + +b + ")";
}
```

#### Output RGB with %s

If we want to return ```rgb()``` using percentages, then we can modify the function to utilize an optional ```isPct``` parameter like so:

```javascript
function hexToRGB(h,isPct) {
  let r = 0, g = 0, b = 0;
  isPct = isPct === true;

  if (h.length == 4) {
    r = "0x" + h[1] + h[1];
    g = "0x" + h[2] + h[2];
    b = "0x" + h[3] + h[3];
    
  } else if (h.length == 7) {
    r = "0x" + h[1] + h[2];
    g = "0x" + h[3] + h[4];
    b = "0x" + h[5] + h[6];
  }
    
  if (isPct) {
    r = +(r / 255 * 100).toFixed(1);
    g = +(g / 255 * 100).toFixed(1);
    b = +(b / 255 * 100).toFixed(1);
  }
  
  return "rgb(" + (isPct ? r + "%," + g + "%," + b + "%" : +r + "," + +g + "," + +b) + ")";
}
```

Under the last ```if``` statement, using ```+```s will convert ```r```, ```g```, and ```b``` to numbers. Each ```toFixed(1)``` along with them will round the result to the nearest tenth. Additionally, we won’t have whole numbers with ```.0``` or the decades old quirk that produces numbers like ```0.30000000000000004```. Therefore, in the ```return```, we omitted the ```+```s right before the first ```r```, ```g```, and ```b``` to prevent ```NaN```s caused by the ```%```s. Now we can use ```hexToRGB("#ff0",true)``` to get ```rgb(100%,100%,0%)```!

### Hex (#rrggbbaa) to RGBA

The procedure for hex values with alpha should again be similar with the last. We simply detect a 4- or 8-digit value (plus ```#```) then convert the alpha and divide it by 255. To get more precise output but not long decimal numbers for alpha, we can use ```toFixed(3)```.

```javascript
function hexAToRGBA(h) {
  let r = 0, g = 0, b = 0, a = 1;

  if (h.length == 5) {
    r = "0x" + h[1] + h[1];
    g = "0x" + h[2] + h[2];
    b = "0x" + h[3] + h[3];
    a = "0x" + h[4] + h[4];

  } else if (h.length == 9) {
    r = "0x" + h[1] + h[2];
    g = "0x" + h[3] + h[4];
    b = "0x" + h[5] + h[6];
    a = "0x" + h[7] + h[8];
  }
  a = +(a / 255).toFixed(3);

  return "rgba(" + +r + "," + +g + "," + +b + "," + a + ")";
}
```

#### Output RGBA with %s

For a version that outputs percentages, we can do what we did in ```hexToRGB()```—switch ```r```, ```g```, and ```b``` to 0–100% when ```isPct``` is ```true```.

```javascript
function hexAToRGBA(h,isPct) {
  let r = 0, g = 0, b = 0, a = 1;
  isPct = isPct === true;
    
  // Handling of digits
  ...

  if (isPct) {
    r = +(r / 255 * 100).toFixed(1);
    g = +(g / 255 * 100).toFixed(1);
    b = +(b / 255 * 100).toFixed(1);
  }
  a = +(a / 255).toFixed(3);

  return "rgba(" + (isPct ? r + "%," + g + "%," + b + "%," + a : +r + "," + +g + "," + +b + "," + a) + ")";
}
```

Here’s a quick fix if the alpha ought to be a percentage, too: move the statement where ```a``` is redefined above the last ```if``` statement. Then in that statement, modify ```a``` to be like ```r```, ```g```, and ```b```. When ```isPct``` is ```true```, ```a``` must also gain the ```%```.

```javascript
function hexAToRGBA(h,isPct) {
  ...
    
  a = +(a / 255).toFixed(3);
  if (isPct) {
    r = +(r / 255 * 100).toFixed(1);
    g = +(g / 255 * 100).toFixed(1);
    b = +(b / 255 * 100).toFixed(1);
    a = +(a * 100).toFixed(1);
  }

  return "rgba(" + (isPct ? r + "%," + g + "%," + b + "%," + a + "%" : +r + "," + +g + "," + +b + "," + a) + ")";
}
```

When we enter ```#7f7fff80``` now, we should get ```rgba(127,127,255,0.502)``` or ```rgba(49.8%,49.8%,100%,50.2%)```.

### RGB to HSL

Obtaining HSL values from RGB or hex is a bit more challenging because there’s a larger formula involved. First, we must divide the red, green, and blue by 255 to use values between 0 and 1. Then we find the minimum and maximum of those values (```cmin``` and ```cmax```) as well as the difference between them (```delta```). We need that result as part of calculating the hue and saturation. Right after the ```delta```, let’s initialize the hue (```h```), saturation (```s```), and lightness (```l```).

```javascript
function RGBToHSL(r,g,b) {
  // Make r, g, and b fractions of 1
  r /= 255;
  g /= 255;
  b /= 255;

  // Find greatest and smallest channel values
  let cmin = Math.min(r,g,b),
      cmax = Math.max(r,g,b),
      delta = cmax - cmin,
      h = 0,
      s = 0,
      l = 0;
}
```

Next, we need to calculate the hue, which is to be determined by the greatest channel value in ```cmax``` (or if all channels are the same). If there is no difference between the channels, the hue will be 0. If ```cmax``` is the red, then the formula will be ```((g - b) / delta) % 6```. If green, then ```(b - r) / delta + 2```. Then, if blue, ```(r - g) / delta + 4```. Finally, multiply the result by 60 (to get the degree value) and round it. Since hues shouldn’t be negative, we add 360 to it, if needed.

```javascript
function RGBToHSL(r,g,b) {
  ...
  // Calculate hue
  // No difference
  if (delta == 0)
    h = 0;
  // Red is max
  else if (cmax == r)
    h = ((g - b) / delta) % 6;
  // Green is max
  else if (cmax == g)
    h = (b - r) / delta + 2;
  // Blue is max
  else
    h = (r - g) / delta + 4;

  h = Math.round(h * 60);
    
  // Make negative hues positive behind 360°
  if (h < 0)
      h += 360;
}
```

All that’s left is the saturation and lightness. Let’s calculate the lightness before we do the saturation, as the saturation will depend on it. It’s the sum of the maximum and minimum channel values cut in half (```(cmax + cmin) / 2```). Then ```delta``` will determine what the saturation will be. If it’s 0 (no difference between ```cmax``` and ```cmin```), then the saturation is automatically 0. Otherwise, it’ll be 1 minus the absolute value of twice the lightness minus 1 (```1 - Math.abs(2 * l - 1)```). Once we have these values, we must convert them to values out of 100%, so we multiply them by 100 and round to the nearest tenth. Now we can string together our ```hsl()```.

```javascript
function RGBToHSL(r,g,b) {
  ...
  // Calculate lightness
  l = (cmax + cmin) / 2;

  // Calculate saturation
  s = delta == 0 ? 0 : delta / (1 - Math.abs(2 * l - 1));
    
  // Multiply l and s by 100
  s = +(s * 100).toFixed(1);
  l = +(l * 100).toFixed(1);

  return "hsl(" + h + "," + s + "%," + l + "%)";
}
```

#### RGB in String

For one string, split the argument by comma or space, strip the ```%```s, and localize ```r```, ```g```, and ```b``` like we did before.

```javascript
unction RGBToHSL(rgb) {
  let sep = rgb.indexOf(",") > -1 ? "," : " ";
  rgb = rgb.substr(4).split(")")[0].split(sep);

  for (let R in rgb) {
    let r = rgb[R];
    if (r.indexOf("%") > -1)
      rgb[R] = Math.round(r.substr(0,r.length - 1) / 100 * 255);
  }

  // Make r, g, and b fractions of 1
  let r = rgb[0] / 255,
      g = rgb[1] / 255,
      b = rgb[2] / 255;

  ...
}
```

### RGBA to HSLA

Compared to what we just did to convert RGB to HSL, the alpha counterpart will be basically nothing! We just reuse the code for RGB to HSL (the multi-argument version), leave ```a``` alone, and pass ```a``` to the returned HSLA. Keep in mind it should be between 0 and 1.

```javascript
function RGBAToHSLA(r,g,b,a) {
  // Code for RGBToHSL(r,g,b) before return
  ...

  return "hsla(" + h + "," + s + "%," +l + "%," + a + ")";
}
```

#### RGBA in String

For string values, we apply the splitting and stripping logic again but use the fourth item in ```rgba``` for ```a```. Remember the new ```rgba(r g b / a)``` syntax? We’re employing the acceptance of it as we did for ```RGBAToHexA()```. Then the rest of the code is the normal RGB-to-HSL conversion.

```javascript
function RGBAToHSLA(rgba) {
  let sep = rgba.indexOf(",") > -1 ? "," : " ";
  rgba = rgba.substr(5).split(")")[0].split(sep);

  // Strip the slash if using space-separated syntax
  if (rgba.indexOf("/") > -1)
    rgba.splice(3,1);

  for (let R in rgba) {
    let r = rgba[R];
    if (r.indexOf("%") > -1) {
      let p = r.substr(0,r.length - 1) / 100;

      if (R < 3) {
        rgba[R] = Math.round(p * 255);
      } else {
        rgba[R] = p;
      }
    }
  }

  // Make r, g, and b fractions of 1
  let r = rgba[0] / 255,
      g = rgba[1] / 255,
      b = rgba[2] / 255,
      a = rgba[3];

  // Rest of RGB-to-HSL logic
  ...
}
```

Wish to leave the alpha as is? Remove the ```else``` statement from the ```for``` loop.

```javascript
for (let R in rgba) {
  let r = rgba[R];
  if (r.indexOf("%") > -1) {
    let p = r.substr(0,r.length - 1) / 100;

    if (R < 3) {
      rgba[R] = Math.round(p * 255);
    }
  }
}
```

### HSL to RGB

It takes slightly less logic to convert HSL back to RGB than the opposite way. Since we’ll use a range of 0–100 for the saturation and lightness, the first step is to divide them by 100 to values between 0 and 1. Next, we find chroma (```c```), which is color intensity, so that’s ```(1 - Math.abs(2 * l - 1)) * s```. Then we use ```x``` for the second largest component (first being chroma), the amount to add to each channel to match the lightness (```m```), and initialize ```r```, ```g```, ```b```.

```javascript
function HSLToRGB(h,s,l) {
  // Must be fractions of 1
  s /= 100;
  l /= 100;

  let c = (1 - Math.abs(2 * l - 1)) * s,
      x = c * (1 - Math.abs((h / 60) % 2 - 1)),
      m = l - c/2,
      r = 0,
      g = 0,
      b = 0;
}
```

The hue will determine what the red, green, and blue should be depending on which 60° sector of the color wheel it lies.

![img](/images/threads/rgb-color-wheel-60.webp)
_The color wheel divided into 60° segments_

Then ```c``` and ```x``` shall be assigned as shown below, leaving one channel at 0. To get the final RGB value, we add ```m``` to each channel, multiply it by 255, and round it.

```javascript
function HSLToRGB(h,s,l) {
  ...

  if (0 <= h && h < 60) {
    r = c; g = x; b = 0;
  } else if (60 <= h && h < 120) {
    r = x; g = c; b = 0;
  } else if (120 <= h && h < 180) {
    r = 0; g = c; b = x;
  } else if (180 <= h && h < 240) {
    r = 0; g = x; b = c;
  } else if (240 <= h && h < 300) {
    r = x; g = 0; b = c;
  } else if (300 <= h && h < 360) {
    r = c; g = 0; b = x;
  }
  r = Math.round((r + m) * 255);
  g = Math.round((g + m) * 255);
  b = Math.round((b + m) * 255);

  return "rgb(" + r + "," + g + "," + b + ")";
}
```

#### HSL in String

For the single string version, we modify the first few statements basically the same way we did for ```RGBToHSL(r,g,b)```. Remove ```s /= 100;``` and ```l /= 100;``` and we’ll use the new statements to wipe the first 4 characters and the ```)``` for our array of HSL values, then the ```%s``` from ```s``` and ```l``` before dividing them by 100.

```javascript
function HSLToRGB(hsl) {
  let sep = hsl.indexOf(",") > -1 ? "," : " ";
  hsl = hsl.substr(4).split(")")[0].split(sep);

  let h = hsl[0],
      s = hsl[1].substr(0,hsl[1].length - 1) / 100,
      l = hsl[2].substr(0,hsl[2].length - 1) / 100;

  ...
}
```

The next handful of statements shall handle hues provided with a unit—degrees, radians, or turns. We multiply radians by 180/π and turns by 360. If the result ends up over 360, we compound modulus divide to keep it within the scope. All of this will happen before we deal with ```c```, ```x```, and ```m```.

```javascript
function HSLToRGB(hsl) {
  ...

  // Strip label and convert to degrees (if necessary)
  if (h.indexOf("deg") > -1)
    h = h.substr(0,h.length - 3);
  else if (h.indexOf("rad") > -1)
    h = Math.round(h.substr(0,h.length - 3) * (180 / Math.PI));
  else if (h.indexOf("turn") > -1)
    h = Math.round(h.substr(0,h.length - 4) * 360);
  // Keep hue fraction of 360 if ending up over
  if (h >= 360)
    h %= 360;
    
  // Conversion to RGB begins
  ...
}
```

After implementing the steps above, now the following can be safely used:

* ```hsl(180 100% 50%)```
* ```hsl(180deg,100%,50%)```
* ```hsl(180deg 100% 50%)```
* ```hsl(3.14rad,100%,50%)```
* ```hsl(3.14rad 100% 50%)```
* ```hsl(0.5turn,100%,50%)```
* ```hsl(0.5turn 100% 50%)```
Whew, that’s quite the flexibility!

### Output RGB with %s

Similarly, we can modify this function to return percent values just like we did in ```hexToRGB()```.

```javascript
function HSLToRGB(hsl,isPct) {
  let sep = hsl.indexOf(",") > -1 ? "," : " ";
  hsl = hsl.substr(4).split(")")[0].split(sep);
  isPct = isPct === true;

  ...

  if (isPct) {
    r = +(r / 255 * 100).toFixed(1);
    g = +(g / 255 * 100).toFixed(1);
    b = +(b / 255 * 100).toFixed(1);
  }

  return "rgb("+ (isPct ? r + "%," + g + "%," + b + "%" : +r + "," + +g + "," + +b) + ")";
}
```

### HSLA to RGBA

Once again, handling alphas will be a no-brainer. We can reapply the code for the original ```HSLToRGB(h,s,l)``` and add ```a``` to the ```return```.

```javascript
function HSLAToRGBA(h,s,l,a) {
  // Code for HSLToRGB(h,s,l) before return
  ...

  return "rgba(" + r + "," + g + "," + b + "," + a + ")";
}
```

#### HSLA in String

Changing it to one argument, the way we’ll handle strings here will be not too much different than what we did earlier. A new HSLA syntax from Colors Level 4 uses ```(value value value / value)``` just like RGBA, so having the code to handle it, we’ll be able to plug in something like ```hsla(210 100% 50% / 0.5)``` here.

```javascript
function HSLAToRGBA(hsla) {
  let sep = hsla.indexOf(",") > -1 ? "," : " ";
  hsla = hsla.substr(5).split(")")[0].split(sep);

  if (hsla.indexOf("/") > -1)
    hsla.splice(3,1);

  let h = hsla[0],
      s = hsla[1].substr(0,hsla[1].length - 1) / 100,
      l = hsla[2].substr(0,hsla[2].length - 1) / 100,
      a = hsla[3];
        
  if (h.indexOf("deg") > -1)
    h = h.substr(0,h.length - 3);
  else if (h.indexOf("rad") > -1)
    h = Math.round(h.substr(0,h.length - 3) * (180 / Math.PI));
  else if (h.indexOf("turn") > -1)
    h = Math.round(h.substr(0,h.length - 4) * 360);
  if (h >= 360)
    h %= 360;

  ...
}
```

Furthermore, these other combinations have become possible:

* ```hsla(180,100%,50%,50%)```
* ```hsla(180 100% 50% / 50%)```
* ```hsla(180deg,100%,50%,0.5)```
* ```hsla(3.14rad,100%,50%,0.5)```
* ```hsla(0.5turn 100% 50% / 50%)```

#### RGBA with %s

Then we can replicate the same logic for outputting percentages, including alpha. If the alpha should be a percentage (searched in ```pctFound```), here’s how we can handle it:

1. If ```r```, ```g```, and ```b``` are to be converted to percentages, then ```a``` should be multiplied by 100, if not already a percentage. Otherwise, drop the %, and it’ll be added back in the ```return```.
2. If ```r```, ```g```, and ```b``` should be left alone, then remove the % from ```a``` and divide ```a``` by 100.

```javascript
function HSLAToRGBA(hsla,isPct) {
  // Code up to slash stripping
  ...
    
  isPct = isPct === true;
    
  // h, s, l, a defined to rounding of r, g, b
  ...
    
  let pctFound = a.indexOf("%") > -1;
    
  if (isPct) {
    r = +(r / 255 * 100).toFixed(1);
    g = +(g / 255 * 100).toFixed(1);
    b = +(b / 255 * 100).toFixed(1);
    if (!pctFound) {
      a *= 100;
    } else {
      a = a.substr(0,a.length - 1);
    }
        
  } else if (pctFound) {
    a = a.substr(0,a.length - 1) / 100;
  }

  return "rgba("+ (isPct ? r + "%," + g + "%," + b + "%," + a + "%" : +r + ","+ +g + "," + +b + "," + +a) + ")";
}
```

### Hex to HSL

You might think this one and the next are crazier processes than the others, but they merely come in two parts with recycled logic. First, we convert the hex to RGB. That gives us the base 10s we need to convert to HSL.

```javascript
function hexToHSL(H) {
  // Convert hex to RGB first
  let r = 0, g = 0, b = 0;
  if (H.length == 4) {
    r = "0x" + H[1] + H[1];
    g = "0x" + H[2] + H[2];
    b = "0x" + H[3] + H[3];
  } else if (H.length == 7) {
    r = "0x" + H[1] + H[2];
    g = "0x" + H[3] + H[4];
    b = "0x" + H[5] + H[6];
  }
  // Then to HSL
  r /= 255;
  g /= 255;
  b /= 255;
  let cmin = Math.min(r,g,b),
      cmax = Math.max(r,g,b),
      delta = cmax - cmin,
      h = 0,
      s = 0,
      l = 0;

  if (delta == 0)
    h = 0;
  else if (cmax == r)
    h = ((g - b) / delta) % 6;
  else if (cmax == g)
    h = (b - r) / delta + 2;
  else
    h = (r - g) / delta + 4;

  h = Math.round(h * 60);

  if (h < 0)
    h += 360;

  l = (cmax + cmin) / 2;
  s = delta == 0 ? 0 : delta / (1 - Math.abs(2 * l - 1));
  s = +(s * 100).toFixed(1);
  l = +(l * 100).toFixed(1);

  return "hsl(" + h + "," + s + "%," + l + "%)";
}
```

### Hex (#rrggbbaa) to HSLA

There aren’t too many lines that change in this one. We’ll repeat what we recently did to get the alpha by converting the hex, but won’t divide it by 255 right away. First, we must get the hue, saturation, and lightness as we did in the other to-HSL functions. Then, before the ending ```return```, we divide the alpha and set the decimal places.

```javascript
function hexAToHSLA(H) {
  let r = 0, g = 0, b = 0, a = 1;

  if (H.length == 5) {
    r = "0x" + H[1] + H[1];
    g = "0x" + H[2] + H[2];
    b = "0x" + H[3] + H[3];
    a = "0x" + H[4] + H[4];
  } else if (H.length == 9) {
    r = "0x" + H[1] + H[2];
    g = "0x" + H[3] + H[4];
    b = "0x" + H[5] + H[6];
    a = "0x" + H[7] + H[8];
  }

  // Normal conversion to HSL
  ...
        
  a = (a / 255).toFixed(3);
                
  return "hsla("+ h + "," + s + "%," + l + "%," + a + ")";
}
```

### HSL to Hex

This one starts as a conversion to RGB, but there’s an extra step to the ```Math.round()```s of converting the RGB results to hex.

```javascript
function HSLToHex(h,s,l) {
  s /= 100;
  l /= 100;

  let c = (1 - Math.abs(2 * l - 1)) * s,
      x = c * (1 - Math.abs((h / 60) % 2 - 1)),
      m = l - c/2,
      r = 0,
      g = 0,
      b = 0;

  if (0 <= h && h < 60) {
    r = c; g = x; b = 0;
  } else if (60 <= h && h < 120) {
    r = x; g = c; b = 0;
  } else if (120 <= h && h < 180) {
    r = 0; g = c; b = x;
  } else if (180 <= h && h < 240) {
    r = 0; g = x; b = c;
  } else if (240 <= h && h < 300) {
    r = x; g = 0; b = c;
  } else if (300 <= h && h < 360) {
    r = c; g = 0; b = x;
  }
  // Having obtained RGB, convert channels to hex
  r = Math.round((r + m) * 255).toString(16);
  g = Math.round((g + m) * 255).toString(16);
  b = Math.round((b + m) * 255).toString(16);

  // Prepend 0s, if necessary
  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;

  return "#" + r + g + b;
}
```

#### HSL in String

Even the first few lines of this function will be like those in ```HSLToRGB()``` if we changed it to accept a single string. This is how we’ve been obtaining the hue, saturation, and lightness separately in the first place. Let’s not forget the step to remove the hue label and convert to degrees, too. All of this will be in place of ```s /= 100;``` and ```l /= 100;```.

```javascript
function HSLToHex(hsl) {
  let sep = hsl.indexOf(",") > -1 ? "," : " ";
  hsl = hsl.substr(4).split(")")[0].split(sep);

  let h = hsl[0],
      s = hsl[1].substr(0,hsl[1].length - 1) / 100,
      l = hsl[2].substr(0,hsl[2].length - 1) / 100;
        
  // Strip label and convert to degrees (if necessary)
  if (h.indexOf("deg") > -1)
    h = h.substr(0,h.length - 3);
  else if (h.indexOf("rad") > -1)
    h = Math.round(h.substr(0,h.length - 3) * (180 / Math.PI));
  else if (h.indexOf("turn") > -1)
    h = Math.round(h.substr(0,h.length - 4) * 360);
  if (h >= 360)
    h %= 360;

  ...
}
```

### HSLA to Hex (#rrggbbaa)

Adding alpha to the mix, we convert ```a``` to hex and add a fourth ```if``` to prepend a 0, if necessary. You probably already familiar with this logic because we last used it in ```RGBAToHexA()```.

```javascript
function HSLAToHexA(h,s,l,a) {
  // Repeat code from HSLToHex(h,s,l) until 3 `toString(16)`s
  ...

  a = Math.round(a * 255).toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;
  if (a.length == 1)
    a = "0" + a;

  return "#" + r + g + b + a;
}
```

#### HSLA in String

Finally, the lines of the single argument version up to ```a = hsla[3]``` are no different than those of ```HSLAToRGBA()```.

```javascript
function HSLAToHexA(hsla) {
  let sep = hsla.indexOf(",") > -1 ? "," : " ";
  hsla = hsla.substr(5).split(")")[0].split(sep);
    
  // Strip the slash
  if (hsla.indexOf("/") > -1)
    hsla.splice(3,1);
    
  let h = hsla[0],
      s = hsla[1].substr(0,hsla[1].length - 1) / 100,
      l = hsla[2].substr(0,hsla[2].length - 1) / 100,
      a = hsla[3];
            
  ...
}
```

### Built-in Names

To convert a named color to RGB, hex, or HSL, you might consider turning [this table of 140+ names and hex values](https://css-tricks.com/snippets/css/named-colors-and-hex-equivalents/) into a [massive object](https://stackoverflow.com/questions/1573053/javascript-function-to-convert-color-names-to-hex-codes#1573141) at the start. The truth is that we really don’t need one because here’s what we can do:

1. Create an element
2. Give it a text color
3. Obtain the value of that property
4. Remove the element
5. Return the stored color value, which will be in RGB by default

So, our function to get RGB will only be seven statements!

```javascript
function nameToRGB(name) {
  // Create fake div
  let fakeDiv = document.createElement("div");
  fakeDiv.style.color = name;
  document.body.appendChild(fakeDiv);

  // Get color of div
  let cs = window.getComputedStyle(fakeDiv),
      pv = cs.getPropertyValue("color");

  // Remove div after obtaining desired color value
  document.body.removeChild(fakeDiv);

  return pv;
}
```

Let’s go even further. How about we change the output to hex instead?

```javascript
function nameToHex(name) {
  // Get RGB from named color in temporary div
  let fakeDiv = document.createElement("div");
  fakeDiv.style.color = name;
  document.body.appendChild(fakeDiv);

  let cs = window.getComputedStyle(fakeDiv),
      pv = cs.getPropertyValue("color");

  document.body.removeChild(fakeDiv);

  // Code ripped from RGBToHex() (except pv is substringed)
  let rgb = pv.substr(4).split(")")[0].split(","),
      r = (+rgb[0]).toString(16),
      g = (+rgb[1]).toString(16),
      b = (+rgb[2]).toString(16);

  if (r.length == 1)
    r = "0" + r;
  if (g.length == 1)
    g = "0" + g;
  if (b.length == 1)
    b = "0" + b;

  return "#" + r + g + b;
}
```

Or, why not HSL? 😉

```javascript
function nameToHSL(name) {
  let fakeDiv = document.createElement("div");
  fakeDiv.style.color = name;
  document.body.appendChild(fakeDiv);

  let cs = window.getComputedStyle(fakeDiv),
      pv = cs.getPropertyValue("color");

  document.body.removeChild(fakeDiv);

  // Code ripped from RGBToHSL() (except pv is substringed)
  let rgb = pv.substr(4).split(")")[0].split(","),
      r = rgb[0] / 255,
      g = rgb[1] / 255,
      b = rgb[2] / 255,
      cmin = Math.min(r,g,b),
      cmax = Math.max(r,g,b),
      delta = cmax - cmin,
      h = 0,
      s = 0,
      l = 0;

  if (delta == 0)
    h = 0;
  else if (cmax == r)
    h = ((g - b) / delta) % 6;
  else if (cmax == g)
    h = (b - r) / delta + 2;
  else
    h = (r - g) / delta + 4;

  h = Math.round(h * 60);

  if (h < 0)
    h += 360;

  l = (cmax + cmin) / 2;
  s = delta == 0 ? 0 : delta / (1 - Math.abs(2 * l - 1));
  s = +(s * 100).toFixed(1);
  l = +(l * 100).toFixed(1);

  return "hsl(" + h + "," + s + "%," + l + "%)";
}
```

In the long run, every conversion from a name becomes a conversion from RGB after cracking the name.

### Validating Colors

In all these functions, there haven’t been any measures to prevent or correct ludicrous input (say hues over 360 or percentages over 100). If we’re only manipulating pixels on a ```<canvas>``` fetched using ```getImageData()```, validation of color values isn’t necessary before converting because they’ll be correct no matter what. If we’re creating a color conversion tool where users supply the color, then validation would be much needed.

It’s easy to handle improper input for channels as separate arguments, like this for RGB:

```javascript
// Correct red
if (r > 255)
  r = 255;
else if (r < 0)
  r = 0;
```

If validating a whole string, then a regular expression is needed. For instance, this is the ```RGBToHex()``` function given a validation step with an expression:

```javascript
function RGBToHex(rgb) {
  // Expression for rgb() syntaxes
  let ex = /^rgb\((((((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?)){2}|((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s)){2})((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]))|((((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){2}|((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){2})(([1-9]?\d(\.\d+)?)|100|(\.\d+))%))\)$/i;

  if (ex.test(rgb)) {
    // Logic to convert RGB to hex
    ...

  } else {
    // Something to do if color is invalid
  }
}
```

To test other types of values, below is a table of expressions to cover both opaque and alpha-enabled:

| Color Value | RegEx |
| ------------- |-------------|
| RGB | ```/^rgb\((((((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?)){2}|((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s)){2})((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]))|((((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){2}|((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){2})(([1-9]?\d(\.\d+)?)|100|(\.\d+))%))\)$/i``` |
| RGBA | ```/^rgba\((((((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?)){3})|(((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){3}))|(((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s){3})|(((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){3}))\/\s)((0?\.\d+)|[01]|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)\)$/i``` |
| Hex | ```/^#([\da-f]{3}){1,2}$/i``` |
| Hex (with Alpha) | ```/^#([\da-f]{4}){1,2}$/i``` |
| HSL | ```/^hsl\(((((([12]?[1-9]?\d)|[12]0\d|(3[0-5]\d))(\.\d+)?)|(\.\d+))(deg)?|(0|0?\.\d+)turn|(([0-6](\.\d+)?)|(\.\d+))rad)((,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}|(\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2})\)$/i``` |
| HSLA | ```/^hsla\(((((([12]?[1-9]?\d)|[12]0\d|(3[0-5]\d))(\.\d+)?)|(\.\d+))(deg)?|(0|0?\.\d+)turn|(([0-6](\.\d+)?)|(\.\d+))rad)(((,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2},\s?)|((\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}\s\/\s))((0?\.\d+)|[01]|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)\)$/i``` |

Looking at the expressions for RGB(A) and HSL(A), you probably have big eyes right now; these were made comprehensive enough to include most of the new syntaxes from CSS Colors Level 4. Hex, on the other hand, doesn’t need expressions as long as the others because of only digit counts. In a moment, we’ll dissect these and decipher the parts. Note that case-insensitive values (```/i```) pass all these.

```javascript
/^rgb\((((((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?)){2}|((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s)){2})((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]))|((((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){2}|((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){2})(([1-9]?\d(\.\d+)?)|100|(\.\d+))%))\)$/i
```

Because ```rgb()``` accepts either all integers or all percentages, both cases are covered. In the outmost group, between the ```^rgb\(``` and ```\)$```, there are inner groups for both integers and percentages, all comma-spaces or spaces only as separators:

1. (((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?){2}|(((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s){2})((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]))
2. ((((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){2}|((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){2})(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)
In the first half, we accept two instances of integers for red and green from 0–99 or 111-199 (```(1?[1-9]?\d)```), 100–109 (```10\d```), 200-249 (```(2[0-4]\d)```), or 250–255 (```25[0-5]```). We couldn’t simply do ```\d{1,3}``` because values like 03 or 017 and those greater than 255 shouldn’t be allowed. After that goes the comma and optional space (,\s?). On the other side of the ```|```, after the first ```{2}``` (which indicates two instances of integers), we check for the same thing with space separators if the left side is false. Then for blue, the same should be accepted, but without a separator.

In the other half, acceptable values for percentages, including floats, should either be 0–99, explicitly 100 and not a float, or floats under 1 with the 0 dropped. Therefore, the segment here is ```(([1-9]?\d(\.\d+)?)|100|(\.\d+))```, and it appears three times; twice with separator (```,\s?){2}```, ```%\s){2}```), once without.

::: danger Hey!
It is legal to use percentages without space separators (```rgb(100%50%10%```) for instance) in CSS, but the functions we wrote don’t support that. The same goes for ```rgba(100%50%10%/50%)```, ```hsl(40 100%50%)```, and ```hsla(40 100%50%/0.5)```. This could very well be a plus for code golfing and minification!
:::

#### RGBA

```javascript
/^rgba\((((((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?)){3})|(((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){3}))|(((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5])\s){3})|(((([1-9]?\d(\.\d+)?)|100|(\.\d+))%\s){3}))\/\s)((0?\.\d+)|[01]|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)\)$/i
```

The next expression is very similar to the pervious, but three instances of integers (```((((1?[1-9]?\d)|10\d|(2[0-4]\d)|25[0-5]),\s?){3})```) or percentages (```(((([1-9]?\d(\.\d+)?)|100|(\.\d+))%,\s?){3})```), plus comma optional space are checked. Otherwise, it looks for the same thing but with space separators, plus a slash and space (```\/\s```) after the blue. Next to that is ```((0?\.\d+)|[01]|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)``` where we accept floats with or without the first 0 (```(0?\.\d+)```), 0 or 1 (```[01]```) on the dot, or 0–100% (```(([1-9]?\d(\.\d+)?)|100|(\.\d+))%```).

#### Hex with Alpha

```javascript
// #rgb/#rrggbb
/^#([\da-f]{3}){1,2}$/i
// #rgba/#rrggbbaa
/^#([\da-f]{4}){1,2}$/i
```

For both hex—with and without alpha—instances of numbers or letters a–f (```[\da-f]```) are accepted. Then one or two instances of this are counted for either short or longhand values supplied (#rgb or #rrggbb). As an illustration, we have this same short pattern: ```/^#([\da-f]{n}){1,2}$/i```. Simply change n to 3 or 4.

#### HSL and HSLA

```javascript
// HSL
/^hsl\((((((\[12]?[1-9]?\d)|[12]0\d|(3[0-5]\d))(\.\d+)?)|(\.\d+))(deg)?|(0|0?\.\d+)turn|(([0-6\\.\d+)?)|(\.\d+))rad)((,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}|(\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2})\)$/i
// HSLA
/^hsla\((((((\[12]?[1-9]?\d)|[12]0\d|(3[0-5]\d))(\.\d+)?)|(\.\d+))(deg)?|(0|0?\.\d+)turn|(([0-6\\.\d+)?)|(\.\d+))rad)(((,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2},\s?)|((\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}\s\/\s))((0?\.\d+)|[01]|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)\)$/i
```

After the ``\(`` in both expressions for HSL and HSLA, this large chunk is for the hue:

```javascript
(((((\[12]?[1-9]?\d)|[12]0\d|(3[0-5]\d))(\.\d+)?)|(\.\d+))(deg)?|(0|0?\.\d+)turn|(([0-6\\.\d+)?)|(\.\d+))rad)
```

```([12]?[1-9]?\d)``` covers 0–99, 110–199, and 210–299. ```[12]0\d``` covers 110–109 and 200–209. Then ```(3[0-5]\d)``` takes care of 300–359. The reason for this division of ranges is similar to that of integers in the ```rgb()``` syntax: ruling out zeros coming first and values greater than the maximum. Since hues can be floating point numbers, the first ```(\.\d+)?``` is for that.

Next to the ```|``` after the aforementioned segment of code, the second ```(\.\d+)``` is for floats without a leading zero.

Now let’s move up a level and decipher the next small chunk:

```javascript
(deg)?|(0|0?\.\d+)turn|((\[0-6\\.\d+)?)|(\.\d+))rad
```

This contains the labels we can use for the hue—degrees, turns, or radians. We can include all or none of ```deg```. Values in ```turn``` must be under 1. For radians, we can accept any float between 0–7. We do know, however, that one 360° turn is 2π, and it stops approximately at 6.28. You may think 6.3 and over shouldn’t be accepted. Because 2π is an irrational number, it would be too messy for this example to try to satisfy every decimal place provided by the JavaScript console. Besides, we have this snippet in our ```HSLTo_()``` functions as a second layer of security if hues 360° or over were to happen:

```javascript
// Keep hue fraction of 360 if ending up over
if (h >= 360)
  h %= 360;
```

Now let’s move up a level and decipher the second chunk:

```javascript
(,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}
```

We’re counting two instances of comma-space-percentages for the saturation and lightness (space optional). In the group after the ```,\s?```, we test for values 0–99 with or without decimal points (```([1-9]?\d(\.\d+)?)```), exactly 100, or floats under 1 without the leading 0 (```(\.\d+)```).

The last part the HSL expression, before the ending (```\)$/i```), is a similar expression if spaces are the only separator:

```javascript
(\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}
```

```\s``` is in the beginning instead of ```,\s?```. Then in the HSLA expression, this same chunk is inside another group with ```,\s?``` after its ```{2}```.

```javascript
((,\s?(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2},\s?)
```

That counts the comma-space between the lightness and alpha. Then if we have spaces as separators, we need to check for a space-slash-space (```\s\/\s```) after counting two instances of space and a percentage.

```javascript
((\s(([1-9]?\d(\.\d+)?)|100|(\.\d+))%){2}\s\/\s))
```

After that, we have this left to check the alpha value:

```javascript
(((0?\.\d+)|[01])|(([1-9]?\d(\.\d+)?)|100|(\.\d+))%)
```

Matches for ```(0?\.\d+)``` include floats under 1 with or without the leading 0, 0 or 1 for ```[01]```, and 0–100%.

### Conclusion

If your current challenge is to convert one color space to another, you now have some ideas on how to approach it. Because it would be tiresome to walk through converting every color space ever invented in one post, we discussed the most practical and browser-supported ones. If you’d like to go beyond supported color spaces (say CMYK, XYZ, or CIE L*a*b*), [EasyRGB](http://www.easyrgb.com/en/math.php)) provides an amazing set of code-ready formulas.

To see all the conversions demonstrated here, I’ve set up a [CodePen demo](https://codepen.io/jkantner/pen/VVEMRK) that shows inputs and outputs in a table. You can try different colors in lines 2–10 and see the complete functions in the JavaScript panel.

<iframe height="500" style="width: 100%;" scrolling="no" title="Color Conversion" src="https://codepen.io/jkantner/embed/VVEMRK?height=265&theme-id=0&default-tab=js,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/jkantner/pen/VVEMRK'>Color Conversion</a> by Jon Kantner
  (<a href='https://codepen.io/jkantner'>@jkantner</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>





## How to use the JavaScript console: going beyond console.log()

_Published: Apr 17, 2019_ by [Yash Agrawal](https://medium.freecodecamp.org/@yagrawl)

Source: [freeCodeCamp](https://medium.freecodecamp.org/how-to-use-the-javascript-console-going-beyond-console-log-5128af9d573b)

One of the easiest ways to debug anything in JavaScript is by logging stuff using ```console.log```. But there are a lot of other methods provided by the console that can help you debug better.

Let’s get started.

The very basic use case is to log a string or a bunch of JavaScript objects. Quite simply,

``` javascript
console.log('Is this working?');
```

Now, imagine a scenario when you have a bunch of objects you need to log into the console.

``` javascript
const foo = { id: 1, verified: true, color: 'green' };
const bar = { id: 2, verified: false, color: 'red' };
```

The most intuitive way to log this is to just ```console.log(variable)``` one after the other. The problem is more apparent when we see how it shows up on the console.

![img](https://cdn-images-1.medium.com/max/1600/1*ZgO6kkyWDThTQRi88QJ8GQ.png)

_No variable names visible_

> As you can see, no variable names are visible. It gets extremely annoying when you have a bunch of these and you have to expand the little arrow on the left to see what exactly the name of the variable is. Enter computed property names. This allows us to basically club all the variables together in a single ```console.log({ foo, bar })``` and the output is easily visible. This also reduces the number of ```console.log``` lines in our code.

### console.table()

We can take this a step further by putting all of these together in a table to make it more readable. Whenever you have objects with common properties or an array of objects use ```console.table()```. Here we can use ```console.table({ foo, bar})``` and the console shows:

![img](https://cdn-images-1.medium.com/max/1600/1*eTNSYg3TJSxDAb5RATys-Q.png)

_console.table in action_

### console.group()

This can be used when you want to group or nest relevant details together to be able to easily read the logs.

This can also be used when you have a few log statements within a function and you want to be able to clearly see the scope corresponding to each statement.

For example, if you’re logging a user’s details:

``` javascript
console.group('User Details');
console.log('name: John Doe');
console.log('job: Software Developer');
// Nested Group
console.group('Address');
console.log('Street: 123 Townsend Street');
console.log('City: San Francisco');
console.log('State: CA');
console.groupEnd();
console.groupEnd();
```

![img](https://cdn-images-1.medium.com/max/1600/1*AnoCROszs2tSyy63x3VaZw.png)

_Grouped logs_

::: tip Default configuration
You can also use ```console.groupCollapsed()``` instead of ```console.group()``` if you want the groups to be collapsed by default. You would need to hit the descriptor button on the left to expand.
:::

### console.warn() & console.error()

Depending on the situation, to make sure your console is more readable you can add logs using ```console.warn()``` or ```console.error()``` . There’s also ```console.info()``` which displays an ‘i’ icon in some browsers.

![img](https://cdn-images-1.medium.com/max/1600/1*q_THCUVbnzpeT7OM952Glw.png)

_warning and error logs_

This can be taken a step further by adding custom styling. You can use a ```%c``` directive to add styling to any log statement. This can be used to differentiate between API calls, user events, etc by keeping a convention. Here’s an example:

``` javascript
console.log('%c Auth ', 
            'color: white; background-color: #2274A5', 
            'Login page rendered');
console.log('%c GraphQL ', 
            'color: white; background-color: #95B46A', 
            'Get user details');
console.log('%c Error ', 
            'color: white; background-color: #D33F49', 
            'Error getting user details');
```

You can also change the ```font-size``` , ```font-style``` and other CSS things.

![img](https://cdn-images-1.medium.com/max/1600/1*t3WewmPmo1GiukJCQfW4dg.png)

_Styling console.log statements_

### console.trace()

```console.trace()``` outputs a stack trace to the console and displays how the code ended up at a certain point. There are certain methods you’d only like to call once, like deleting from a database. ```console.trace()``` can be used to make sure the code is behaving the way we want it to.

### console.time()

Another important thing when it comes to frontend development is that the code needs to be fast. ```console.time()``` allows timing of certain operations in the code for testing.

``` javascript
let i = 0;
console.time("While loop");
while (i < 1000000) {
  i++;
}
console.timeEnd("While loop");
console.time("For loop");
for (i = 0; i < 1000000; i++) {
  // For Loop
}
console.timeEnd("For loop");
```

![img](https://cdn-images-1.medium.com/max/1600/1*yJAlR3bKkBhpFY5MNB8gQA.png)

_console.time() output for loops_

Hopefully, the article provided some information on various ways to use the console. If you have any questions or want me to elaborate please leave me a note below or reach out to me on [twitter](http://twitter.com/yagrawl) or [email](yagrawl2@gmail.com).