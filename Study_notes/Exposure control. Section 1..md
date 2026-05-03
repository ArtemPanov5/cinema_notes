Exposure control. Section 1.

---
Guidelines:

- Concept of exposure control
- Tasks of exposure control
- Adams' Zone System
- Through the lens (TTL) system
- Creating stability and reproducibility of the through cinematographic process (TCP) of film type
---

Task of Exposure Control

When working with black-and-white images, we can reduce the task of color reproduction to the task of correct transmission of object luminances. When we work with color images, we need to correctly transmit tone (luminance) and chromaticity (hue and saturation). The task of chromaticity transmission cannot be correctly resolved without proper luminance transmission, since hue and saturation lie on a cross-section along the luminance axis in the Ostwald and Munsell color solids.

Two aspects of tone reproduction:
- objective - technical correspondence of tone relationships in the object and in the image
- subjective - similarity of psychophysiological reactions when perceiving the object and its image

No visible color can be reproduced on screen as similar unless, as a result of correct exposure, it is placed in the image at the same brightness level at which we perceived it in the object. After all, color reproduction is color separation and synthesis at a certain luminance level within highlights, lights, and shadows.

The algorithm for correct exposure must, first of all, take into account the characteristics of psychophysiological perception of the object and then its image. We should select such a level of luminance adaptation of the TCP (through cinematographic process) that would correspond to the luminance adaptation level of our eye when viewing the object—only in this way can we achieve similarity of psychophysiological reactions when perceiving the object and its image.

The task of exposure control is to give us an understanding of how the contrast of the shooting object relates to the magnitude of Optimal Visual Contrast (OVC), in order to then determine what degree of luminance adaptation of the system, taking into account the adaptation of our vision when viewing the shooting object.

---

Adams' Zone System

The theoretical basis for exposure control is still Adams' Zone System, which considers any shooting object as a series of different zones with different brightness. table[1]

Adams divided the part of the brightness range that our eye is capable of perceiving into 10 zones available for TCP reproduction, each of which differs in brightness from the previous one by exactly two times. OVC is a window accommodating 5 such zones at a time. And we must, as it were, fit this window on Adams' scale in the same way as our eye adapts when perceiving the object.

$D$ - optical density of negative
$R$ or $T$ - reflection/transmittence (reflection coefficient, transmittance coefficient)

$$D = log_{10}(\frac{1}{R})$$
In Adams' zone theory, each zone differs from the adjacent one in brightness by a factor of 2, which gives $\Delta{D} = log_{10}(2) \approx 0.301$.

Example: illustration[1]

From the table of reflectance of various surfaces, let us take two objects that have a twofold difference in reflection coefficient values.

Let us take:
- $R_1$(fresh snow) $\approx$ 0.8,
- $R_2$(old wet snow) $\approx$ 0.4.

Next, we calculate using the formula above the optical density for each object:
$$D_1 = log_{10}(\frac{1}{R_1}) = log_{10}(\frac{1}{0.8}) \approx 0.097$$
$$D_2 = log_{10}(\frac{1}{R_2}) = log_{10}(\frac{1}{0.4}) \approx 0.398$$

Thus $\Delta{D} = |D_1-D_2| = |0.097 - 0.398|\approx 0.301$. We saw that indeed $\Delta{D} \approx 0.301$ when comparing two surfaces that have a twofold difference in reflection coefficient values. The same applies to T (transmittance coefficient).

Moreover, $\Delta{D} \approx 0.301$ corresponds to an exposure change of 1EV (relative exposure value), or 1 stop change in shutter speed, aperture, or ISO values.

1 stop is equivalent to a change in values:

- shutter speed: $1/125 \rightarrow 1/60$
- aperture: $f/4 \rightarrow f/2.8$
- ISO: ISO 200 $\rightarrow$ ISO 400

etc.

---

Through the lens (TTL) system

The Through the lens (TTL) metering system, in accordance with its name, performs measurement directly through the entire optical system (lens, optical attachment/optical filter). Thus, such a system automatically takes into account optical system characteristics such as light transmission[2] and light scattering[3].

Early TTL systems.

The first "TTL" systems could only measure integral brightness in the viewing field, so it was necessary to introduce a system of corrections, sometimes quite significant, especially in cases when the object's contrast differed greatly from the optimal visual contrast value.

"MID TONE"/"key"/"grey field" - a gray field with a reflection coefficient of 0.18 (albedo). Such a gray field is an analog of a shooting object that has a brightness range corresponding to OVC, that is, 1:40. 0.18 is the arithmetic mean of the entire brightness series in the object.

The algorithm of early systems was as follows:

1. Measurement of object brightnesses in the frame is performed.
2. The arithmetic mean brightness of all objects is calculated.
3. The obtained number is compared with "MID TONE".
4. Based on this comparison, the device gives recommendations for changes in shooting parameter values: aperture, shutter speed, ISO.

Early TTL systems provide sufficiently accurate transmission of any object whose contrast does not exceed the OVC value. If the object's contrast in the viewing field is greater than OVC, then corrections must be resorted to, which are quite subjective, as they depend on which part of the frame is currently plot-important for us.

Spotmeter - a device for spot measurement of brightness on the shooting object. They have a photometry angle of 1 degree.

---

Creating stability and reproducibility of the through cinematographic process (TCP) of film type

In cinema, for image construction we use only the straight section of the negative's characteristic curve—an essential condition for a good negative. According to standards, the beginning of this section is defined as D(fog)[4] + 0.12 - 0.15, black with texture. Closer to the upper edge of the negative's characteristic straight line is located the section with the highest density D(fog)+ 2.8 - 3, white with texture. Optical density D(fog) + 0.7 is middle gray 18%.

Negative gradient during development is the coefficient of negative density change from one gray field section to another, adjacent sections have a brightness difference of 2 times. The negative gradient according to regulations is in the range from 0.51 to 0.65. The formula for calculating the optical density gradient is as follows: $$\Delta D_{negative} = 0.301 * \gamma_{negative}$$
Example:

When developing films No. 1 and No. 2, $\gamma_{negative_1} = 0.51$ and $\gamma_{negative_2} = 0.65$ are used respectively. That is, in accordance with the formula above, we have $\Delta D_{negative_1} \approx 0.15$ and $\Delta D_{negative_2} \approx 0.2$. We obtain the following densities in the negative after development:

| Film sample \ Gray zone | 1   | 2    | 3   | 4    | 5   | 6    |
| --------------------------- | --- | ---- | --- | ---- | --- | ---- |
| No. 1                          | 0.2 | 0.35 | 0.5 | 0.65 | 0.8 | 0.95 |
| No. 2                          | 0.2 | 0.4  | 0.6 | 0.8  | 1.0 | 1.2  |


An increase in negative gradient occurs as a result of longer exposure of the developer solution to the material. It can be said that this is a natural side effect of longer development, which is used to increase film sensitivity (from the table we see that in section No. 4 the negative density in film No. 2 is almost a full stop higher than in film No. 1). This works with varying degrees of effectiveness, but it must be remembered that an increase in the developed negative gradient above the range of values from 0.51 to 0.65 will lead to the impossibility of printing all densities onto positive film, whose latitude is significantly inferior to the width of negative film. Therefore, such a technique for increasing sensitivity is undesirable and should be used with caution, or in situations where the material was deliberately underexposed but is very valuable and needs to be preserved. In color processes, gradient change is unacceptable because it violates the stability and reproducibility of TCP results.

When printing to positive, on the contrary, the "ends" of the characteristic curve are used. Thus we have two gradients: $\gamma_{positive_1} \in [D_{fog} ; D = 1.0]$, $\gamma_{positive_2} \in [D = 1.0 ; D_{top}]$. D = 1.0 is the optical density of middle gray 18%; by this parameter the aperture is adjusted during shooting, and also the light number in the printing machine is determined.

To create tonal unity with different types of film, all used film must have very close gradient values to each other, preferably close to the value of 0.51. When using black-and-white newsreel footage in a color film, the newsreel must be duplicated onto color negative. When black-and-white episodes are present, the black-and-white negative must be developed to the color negative gradient level. In such a case, when shooting a black-and-white episode, it must be shot 1 stop higher than color to compensate for the optical density added by the mask in the color negative.

For stability and reproducibility of results when testing film, it is necessary to determine such an exposure to obtain the required densities in the negative (this depends on illumination strength (brightness), film sensitivity, processing gradient, lens relative aperture, shutter angle, and shooting frame rate). Then during shooting, one should strive to ensure that tonal relationships are always transmitted in the negative at the same density levels, and also that the reference point is at the density level D(fog) + 0.7.

In practice during shooting, one operates not with densities in the negative, but with the accuracy of gray transmission in each zone of an equal-step scale—an analog of an image reduced to OVC.

If accurate gray transmission is ensured at all levels of such a scale, then the main condition for correct transmission of the object's subject colors is such placement of these colors at the exposure level that corresponds to the visual perception of these colors in the object by luminance. In this case, we will achieve psychological similarity of the filmed object and image.

Conscious deviations from the rules of "good" positive and negative also take place, as tools of creative choice. These deviations imply shifting exposure up or down in different episodes.

| OVC Zone | Stops from key | Relative brightness | Reflection coeff. (ρ) | Negative density | Visual perception | Typical frame examples                        |
| -------- | ----------------- | --------------------- | -------------------- | ------------------ | --------------------- | ----------------------------------------------- |
| +2       | +2 stops          | 4                     | ~0.90                | D₀ + 2.8-3.0       | White with texture      | White suit, white shirt, bright highlights        |
| +1       | +1 stop           | 2                     | ~0.63                | D₀ + 1.8-2.0       | Very light         | Lights on well-lit face, light skin   |
| 0        | 0 (Mid-Tone)      | 1                     | ~0.18                | D₀ + 0.7           | Medium gray (18%)    | Light halftones on face, reflections, gray card |
| -1       | -1 stop           | 0.5                   | ~0.09                | D₀ + 0.5           | Dark                | Halftones/shadows on face, dark areas           |
| -2       | -2 stops          | 0.25                  | ~0.045               | D₀ + 0.3           | Very dark          | Deep shadows on face, dark folds           |
| -3       | -3 stops          | 0.125                 | ~0.023               | D₀ + 0.12-0.15     | Black with texture     | Dense pure shadows, black fabric with texture     |

---

In a color image, for correct color reproduction without distortion, it should be reproduced at the same luminance level at which it is visually perceived. Thus, for example, for accurate transmission of the most saturated tones of different chromaticity, they should be placed in different zones: yellow in the zone between key and white, violet in the zone between key and black, and red and green in the key zone (recall Munsell's color solid).

So, two tasks of exposure calculations (the specified order of their solution is important):
1. artistic
2. engineering-technical

All cases of deviations from the above-stated rules of the technological process must be motivated by an artistic task.

Example:

We observe red, orange, and yellow cars driving on a wet road from a window. Our vision adapts so that the brightness of the cars in our field of vision is between key and white, where these colors appear most saturated.

If we were looking at this picture from the ground, for example from a roadside, and saw the cars against an overcast sky, which nevertheless significantly exceeds other objects in our field of vision in brightness, then our visual analyzer would adapt differently, placing the sky in the area between key and white, and colored areas below the key. This would lead to the colors no longer being perceived by us as bright and saturated.

If in the second case we still need to transmit the color as saturated, then we can open the aperture so much (or change other system parameters) that the colored areas are again in the zone between key and white. Yes, in this case we will get overexposure in the sky area, but if it is more important for us to transmit the color of the cars, then we can justifiably sacrifice texture in the sky.

---

Literature:

- "Zheleznyakov V.N. Color and Contrast. Technology and Creative Choice. Moscow: VGIK, 2001"

---

Notes and tables:

[1] Table with representation of an equal-step brightness series, expressed in relative exposure values ("EV"), and next to it the corresponding values of standard brightness units ("candela per square meter" and "foot-lambert").

| Zone | EV  | Brightness (cd/$m^2$) | Brightness (foot-lambert) | Description                               | Examples                                      |
| ---- | --- | --------------------- | ------------------------- | ----------------------------------------- | --------------------------------------------- |
| 0    | -5  | <0.3                  | <0.1                      | Absolutely black tone without details     | Deep shadow, unlit openings                   |
| I    | -4  | 0.3–0.6               | 0.1–0.2                   | Almost black, details not distinguishable | Deep shadows, close to black                  |
| II   | -3  | 0.6–1.2               | 0.2–0.35                  | First signs of texture in shadows         | Dark fur, textiles in deep shadow             |
| III  | -2  | 1.2–2.5               | 0.35–0.7                  | Dark tones with detail rendering          | Dark foliage, tree bark, shadows on clear day |
| IV   | -1  | 2.5–5                 | 0.7–1.5                   | Medium shadows, dark foliage              | Wet grass, tanned skin, dark clothing         |
| V    | 0   | 5–10                  | 1.5–3                     | Medium gray (18% reflection)              | Gray card, average skin tone, asphalt         |
| VI   | +1  | 10–20                 | 3–6                       | Light midtones                            | Caucasian skin, light wood                    |
| VII  | +2  | 20–40                 | 6–12                      | Light tones with rendering                | Light plaster, paper, clouds with texture     |
| VIII | +3  | 40–80                 | 12–24                     | Very light tones, texture preserved       | Snow with texture, white fabric with details  |
| IX   | +4  | 80–160                | 24–48                     | Almost white, minimum details             | Shining snow, reflections on chrome           |
| X    | +5  | >160                  | >48                       | Pure white without details                | Sun reflections, light sources                |

[2] Light transmission of optical system:
1. Geometric light transmission (in practice - synonym of F-stop), a value showing how many times the illumination in the focal plane is less than the brightness of the shooting object.$$Q_g = (D/f')^2$$
   $D$ - aperture diameter,
   $f'$ - focal distance.
   
   In practice, usually understood as the maximum aperture number of the lens, that is, through the F-number, calculated as:$$F = f'/D$$
   Does not take into account light losses when passing through
   optical lens groups.
2. Effective light transmission (T-stop) takes into account light losses when passing through optical lens groups. Cinema lenses more often indicate precisely this value.

[3] Light scattering of optical system - unwanted light scattering inside the lens, leading to reduced contrast and the appearance of flares. Quantitatively described by the light scattering coefficient (R), which is measured experimentally for a specific lens. Cinema lenses have $R \approx 1-3$%.

[4] D(negative fog) - optical density of unexposed and developed negative without taking into account the optical density of the base.

---

Illustrations:

[1] Graph of negative density function values depending on object brightness.
![[IMG_20251116_200354_509~2.png]]

---
