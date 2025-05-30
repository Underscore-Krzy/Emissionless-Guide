---
layout: default
---

## Step 1:
Remove the gradients from both your broadband and narrowband images. Though it's not strictly necessary, you can also color calibrate your broadband at this step. Keep the stars in for now, and don't perform any sharpening before subtracting the emissions to ensure accurate subtraction. Don't denoise your broadband and avoid denoising your narrowband if you can help it. Do not stretch the image yet - both continuum and emission subtraction need to be done on linear data. 
* [OSC ONLY] You need to extract all emission bands from your dualband images. Due to how the filters used in the CFA pass light, the G and B channels will contain some red signal, meaning that, if you're using an Ha/Oiii dualband filter, the Oiii will contain some Ha "leakage". To remedy this you can use the [DBXtract sctipt](https://dbxtract.astrocitas.com/) but I usually get more accurate results by doing it manually. Use the standard PixelMath for image subtraction `$T - factor * ($T[0] - med($T[0]))` on both G and B channels. Find the highest factors that still avoid oversubtraction. If an emission band is captured in two color channels simultaneously (e.g. Oiii), combine them. Tip: Oiii combination factors are about 66% G and 33% B when combining.

<div id="carouselStep1" class="carousel slide" data-bs-ride="carousel">
  <div class="carousel-inner">
    <div class="carousel-item active">
      <img src="assets/dualband - properly subtracted.jpg" class="d-block w-100" alt="Properly subtracted Oiii channel" />
    </div>
    <div class="carousel-item">
      <img src="assets/dualband - oversubtracted.jpg" class="d-block w-100" alt="Oversubtracted Oiii channel" />
    </div>
    <div class="carousel-item">
      <img src="assets/dualband - original.jpg" class="d-block w-100" alt="Original Oiii channel" />
    </div>
  </div>
  <button class="carousel-control-prev" type="button" data-bs-target="#carouselStep1" data-bs-slide="prev">
    <span class="carousel-control-prev-icon"></span>
    <span class="visually-hidden">Previous</span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#carouselStep1" data-bs-slide="next">
    <span class="carousel-control-next-icon"></span>
    <span class="visually-hidden">Next</span>
  </button>
</div>
###### Figure 1. Properly subtracted vs oversubtracted vs original Oiii channel of the Pacman nebula with a dualband filter. Notice the telltale sign of oversubtracting - previously bright areas are now darker than the image median/background.

## Step 2:
If possible, identify a feature that's visible exclusively (or primarily in one of the emission bands you’ve captured, and that also shows up clearly in the broadband data. This will make the subsequent steps easier.

<Carousel>
  <img src="assets/Structure pick - Ha.jpg" alt="Ha" />
  <img src="assets/Structure pick - O III.jpg" alt="O III" />
  <img src="assets/Structure pick - RGB.jpg" alt="RGB" />
  <img src="assets/Structure pick - Ha annotated.jpg" alt="Ha annotated" />
</Carousel>
###### Figure 2. Example of how to choose the ideal reference structure

## Step 3:
Now comes the fun part. You need to increase the subtraction factors for the emission band you've chosen in step 2. The PixelMath used to make an emissionless image is the standard image subtraction, just with more images.
Emissionless PixelMath for SHO:
```js
// R:
$T - r_h * (Ha - med(Ha)) - r_o * (Oiii - med(Oiii)) - r_s * (Sii - med(Sii))

// G:
$T - g_h * (Ha - med(Ha)) - g_o * (Oiii - med(Oiii)) - g_s * (Sii - med(Sii))

// B:
$T - b_h * (Ha - med(Ha)) - b_o * (Oiii - med(Oiii)) - b_s * (Sii - med(Sii))

// Symbols:
r_h = 0.0,
r_o = 0.0,
r_s = 0.0,

g_h = 0.0,
g_o = 0.0,
g_s = 0.0,

b_h = 0.0,
b_o = 0.0,
b_s = 0.0
```
Tweak the symbols for the selected band until the feature you identified in Step 2 disappears or there are signs of oversubtraction.
Some general tips:
* My estimates of **relative** subtraction factors for SHO (keep in mind these are only there to give you a starting point):
  * Sii - `R = 1.0, G = 0.0, B = 0.0`
  * Ha - `R = 1.0, G = 0.2, B = 0.3`
  * Oiii - `R = 0.0, G = 0.9, B = 1.0`
* Always inspect both the individual channels and the combined RGB image after each change. It's easier to notice oversubtraction in grayscale and discoloration in color images.
* Remember, there's a lot more emission lines than just Sii, Ha, and Oiii. Sometimes, if you can't get some structure to disappear without oversubtracting another, it might just be because it's made up of tons of different spectral lines. It's fine if that happens - making a truly emissionless image is practically impossible.
* With some targets, if you only have Ha and Oiii data, it might be beneficial to make an Sii estimate by treating it as Ha minus Oiii. In that case you can just set the Oiii factor to negative and increase the Ha factor. It's not often that I'd recommend doing this, however - 9 times out of 10 the residual Sii emission will usually be insignificant once you reintroduce the narrowband.

## Step 4:
Repeat steps 2 and 3 until you're left with just one emission channel to subtract. Increase its subtraction factors until you see signs of oversubtraction.
More often than not you'll need to tweak the subtraction factors you chose previously for other emission bands, especially when you're subtracting more than one from a single color channel.

<Carousel>
  <img src="assets/emission subtraction - RGB.jpg" alt="RGB image after subtraction" />
  <img src="assets/emission subtraction - Ha subtracted.jpg" alt="Ha subtracted" />
  <img src="assets/emission subtraction - Initial emissionless.jpg" alt="Initial emissionless" />
  <img src="assets/emission subtraction - Final tweaks.jpg" alt="Final tweaks" />
</Carousel>
###### Figure 3. Stages of an example emission subtraction. First, I subtracted Ha, then Oiii. I tweaked the factors slightly for the G and B channels at the end to try to subtract more emission. I recommend viewing the individual channels of these images to get a better understanding of what's going on.

## Example usage
Generally, emissionless processing is most useful when there is bright emission mixed with broadband components like dust or reflection nebulae. Take the Flaming Star Nebula for example - there's a bright reflection nebula you could boost and a ton of dust hidden by the surrounding Ha you could reveal by separating the image components. Emissionless processing is less useful when the emission isn't very bright relative to the features you want to reveal, like pretty much every galaxy.
There are a couple of ways to use emissionless images but the main three are:
1. A simple non-emission enhancement - stretch the emissionless image like usual, though you probably want to keep it on the darker, understretched side. Screen blend `~(~$T * ~emissionless)` it with your final image towards the end of your processing, similar to how you'd do narrowband addition.
2. A base for narrowband addition - instead of adding the narrowband to your broadband image you instead add it to the emissionless image. This allows for more precise control over each part of the image, especially when paired with HDR stretching of the narrowband channels. This way, you'll preserve the features that would be otherwise overshadowed by bright emission while also adding the faint narrowband features.
3. A better continuum reference - technically continuum subtraction should be done using emissionless images. While it might not be strictly necessary for Ha or Oiii continuum subtraction, Sii will very often get severely oversubtracted if you don't remove Ha emission from the red channel before CS.
   
<Carousel>
  <img src="assets/addition - before.jpg" alt="Before emissionless enhancement" />
  <img src="assets/addition - after.jpg" alt="After emissionless enhancement" />
</Carousel>
###### Figure 4. Example of using the emissionless image to enhance the continuum structures

<Carousel>
  <img src="assets/Emissionless as base - before.jpg" alt="Before using emissionless as base" />
  <img src="assets/Emissionless as base - after.jpg" alt="After using emissionless as base" />
</Carousel>
###### Figure 5. Example of using the emissionless image as a base for narrowband addition
