# cga-ypbpr
 CGA/Tandy/EGA/Hercules to component video converter

![Two units stacked](/img/2ofthem.jpg)
Two units stacked, showing both sides

## Overview
Pre-VGA PCs use a handful of distinct video card types (CGA, Tandy, EGA, Hercules, MDA) to produce video. These have specific (but related) signaling systems, and all share a 9-pin D-Sub connector. This device converts these video signals into analog component video.

## Principle of operation
The above video systems use simple TTL digital signaling, which can be interpreted with gate logic (in this case, a GAL chip). Resistors are used to form a simple DAC for the analog outputs. This GAL has 10 outputs, providing 4 bits of precision for the Y (luminance) channel, and 3 bits each for Pb (blue) and Pr (red). This is the same basic concept as [Necroware's design](https://github.com/necroware/mce-adapter) and the [GGlabs CGA2RGB](https://gglabs.us/node/2284), just with YPbPr instead of RGBS output. Like these, it does format conversion but *not* scan conversion - the output signal has the same video timings as the input. A pulse stretching circuit adds a blank area to the beginning of each scanline, so that the TV (or scan converter) can set its black level, even if the PC is displaying a border. A simple lowpass filter provides detection of sync polarity, allowing the logic to autoswitch between CGA and other modes. Mono and EGA high-res modes can't be distinguished by polarity alone, so a switch is provided to select between them.

### Why component video?
It seems to be absent from the usual conversion options, with most converters of this type converting to 15.7kHz VGA (which is hard to accommodate since most monitors won't sync this) or SCART (which isn't common outside of Europe). Component, on the other hand, can be used directly with many TVs (CRT *and* LCD). It can also be run into scan converters, which commonly take this format. In addition, high quality cables are cheap and readily available for component video.

### So, about mono and high-res EGA...
This is where things get a little less "it just works". Unfortunately, neither of these produce video timings that are normal for component video. Tests with various component devices has yielded mixed results (although the [OSSC](https://videogameperfection.com/products/open-source-scan-converter/) is happy to work with this). 200-line modes worked in all cases, as expected (CGA uses standard NTSC video timings).

### I didn't understand any of that. What's this mean to me?
It means that you can connect any CGA, Tandy, or EGA source to any TV with a component video input. You can't use a VGA monitor (unless you add a scan converter). For EGA to work with all displays, you'll need to set the card to always use 200-line video modes. On most EGAs, this is done by setting the first 4 DIP switches to Off, Off, Off, On.

### Can I connect a Commodore 128?
Probably? I don't have a C128. These use the same signaling and pinout as CGA, so they should work.

## Power and cabling
Required power is 9V DC through a 2.1x5.5mm barrel jack, center positive. Any current/power rating should work - the device draws, at worst, 135mA (when using the non-ZeroPower version of the ATF22V10). With any luck, you already have a power brick with this voltage and connector. If you don't, you can [add one](https://www.mouser.com/ProductDetail/552-AA03A-090A-R) to your Mouser order. Component cables can be easily sourced, and 3 composite video cables are usable in a pinch. The required PC video cable is a male-to-male D-Sub DE-9 cable (all pins wired straight through). This can be [bought from Serdaco](https://www.serdashop.com/9-Pin-video-cable-2m) or built. To build one, you'll need two [DE-9 male plugs](https://www.mouser.com/ProductDetail/523-L717SDE09P), two [backshells](https://www.mouser.com/ProductDetail/156-2009-EX), and some shielded 9-core cable (connect the shield to pin 1). A modem cable can be used, but you'll probably need a gender changer as these are usually male-female. A *null* modem cable ***will not work*** as they have pins swapped.

![Unit interior](/img/interior.jpg)

## Assembly
The device is designed to fit in a common Hammond 1593K enclosure. The project provides 3 PCBs - the main board, and two that serve as the side panels. These should all be ordered at the standard 1.6mm thickness (if you have any colour preference for the side panels, remember to select it while ordering). With boards and components in hand, you can go ahead and solder everything down. Install the power LED "floating" above the board so it can be pushed into the hole in the panel, and mount the trimmer pot as high as its legs will allow to make adjustment easier. Once all components have been soldered into the main board, slide the side panels on over the jacks and switches, and lower the whole assembly into the lower case half, with the panels aligned to the case grooves. Don't install the chip in the socket yet though - we need to program it first.

### Programming the GAL
The ATF22V10 can be programmed in an EPROM programmer, like a TL866 [or T48](https://xgecu.myshopify.com/collections/xgecu-t48-tl866ii-3g-programmer). Source and compiled output are in the `pld` folder. Programming the chip just needs the `cga-ypbpr.jed` file.

### (optional) Editing the GAL source
The source `cga-ypbpr.pld` can be edited in WinCUPL, which is downloaded for free [from Microchip](https://www.microchip.com/en-us/products/fpgas-and-plds/spld-cplds/pld-design-resources). The following compiler options (under Options → Compiler) are required to build it:

- Minimization: Select **Quine-McCluskey**
- Optimization: Check **Best For Polarity**, uncheck all others

### Adjusting the trimmer potentiometer
The trimmer adjusts how long the blanking period applied to the beginning of each scanline is. This blanking is only applied in 200-line modes (the fat "CGA border" is unique to these modes); for EGA cards, see the note above about EGA DIP switches. Start by turning the trimmer all the way to the minimum position (counter-clockwise), then power the unit (and suitable old computer) on. Next, load up a game that uses a border (Keen 4 works well), or enter `COLOR , , 5` in GW-Basic/QBasic. The screen should visibly dim (and turn the wrong colour), due to the video decoder in the TV or scan converter "seeing" the border where there should be black. Now turn the trimmer clockwise and the screen will get brighter - stop turning when it no longer has an effect on brightness. You can turn it further, but a black bar will appear on the left hand side and start covering part of the screen. Many EGA cards insert their own blanking, and don't dim even in the leftmost position (very nice!); for these, set the trimmer so the black bar is just offscreen.

## Parts list
Bill Of Materials and part references are below. The specified parts are just the ones I used, and can be substituted as needed - Mouser links provided for convenience and reference. The case is available in other colours!

| Reference | Value | Qty | Mouser link |
| --------- | ----- | --- | ----------- |
| C1-C3 | 1μF ceramic | 3 | [KEMET C315C105K3R5TA](https://www.mouser.com/ProductDetail/80-C315C105K3R) |
| C4 | 100μF electrolytic | 1 | [Panasonic ECA-1VM101I](https://www.mouser.com/ProductDetail/667-ECA-1VM101I) |
| C5-C7 | 47pF ceramic | 3 | [KEMET C315C470K2G5TA](https://www.mouser.com/ProductDetail/80-C315C470K2G) |
| C8 | 1nF ceramic | 1 | [KEMET C315C102K1R5TA](https://www.mouser.com/ProductDetail/80-C315C102K1R) |
| D1 | 3mm green LED | 1 | [Lite-On LTL-4231N](https://www.mouser.com/ProductDetail/859-LTL-4231N) |
| FB1 | Ferrite bead | 1 | [Wurth 7427502](https://www.mouser.com/ProductDetail/710-7427502) |
| J1 | D-Sub DE-9 female | 1 | [Amphenol L77SDE09SA4CH4F](https://www.mouser.com/ProductDetail/523-L77SDE09SA4CH4F) |
| J2 | 2.1x5.5mm barrel | 1 | [CUI PJ-002A](https://www.mouser.com/ProductDetail/490-PJ-002A) |
| J3 | RCA, green | 1 | [Kycon KLPX-0848A-2-GR](https://www.mouser.com/ProductDetail/806-KLPX-0848A-2-GR) |
| J4 | RCA, blue | 1 | [Kycon KLPX-0848A-2-BL](https://www.mouser.com/ProductDetail/806-KLPX-0848A-2-BL) |
| J5 | RCA, red | 1 | [Kycon KLPX-0848A-2-R](https://www.mouser.com/ProductDetail/806-KLPX-0848A-2-R) |
| Q1, Q2 | 2N7000 | 2 | [onsemi 2N7000](https://www.mouser.com/ProductDetail/512-2N7000BU) |
| R1, R6, R10 | 732Ω 1% | 3 | [YAGEO MFR-25FRF52-732R](https://www.mouser.com/ProductDetail/603-MFR-25FRF52-732R) |
| R2, R7, R11 | 1k43Ω 1% | 3 | [YAGEO MFR-25FRF52-1K43](https://www.mouser.com/ProductDetail/603-MFR-25FRF521K43) |
| R3, R8, R12 | 2k87Ω 1% | 3 | [YAGEO MFR-25FRF52-2K87](https://www.mouser.com/ProductDetail/603-MFR-25FRF522K87) |
| R4, R9, R13 | 150Ω 1% | 3 | [YAGEO MFR-25FTF52-150R](https://www.mouser.com/ProductDetail/603-MFR-25FTF52-150R) |
| R5 | 365Ω 1% | 1 | [YAGEO MFR-25FRF52-365R](https://www.mouser.com/ProductDetail/603-MFR-25FRF52-365R) |
| R14, R17 | 10kΩ 5%| 2 | [YAGEO CFR-25JB-52-10K](https://www.mouser.com/ProductDetail/603-CFR-25JB-52-10K) |
| R15 | 330Ω 5% | 1 | [YAGEO CFR-25JB-52-330R](https://www.mouser.com/ProductDetail/603-CFR-25JB-52-330R) |
| R16 | 1kΩ 5% | 1 | [YAGEO CFR-25JR-52-1K](https://www.mouser.com/ProductDetail/603-CFR-25JR-521K) |
| RV1 | 10k linear | 1 | [Piher PT10LH01-103A](https://www.mouser.com/ProductDetail/531-PT10H-10K) |
| SW1 | SPDT toggle | 1 | [Grayhill 34CMSP12B2M7QT](https://www.mouser.com/ProductDetail/706-34CMSP12B2M7QT) |
| SW2 | SPDT slide | 1 | [C&K OS102011MA1QN1](https://www.mouser.com/ProductDetail/611-OS102011MA1QN1) |
| U1 | MCP1702-500 | 1 | [Microchip MCP1702-5002E](https://www.mouser.com/ProductDetail/579-MCP1702-5002E/TO) |
| U2 | ATF22V10C | 1 | [Atmel ATF22LV10CQZ-30](https://www.mouser.com/ProductDetail/556-AF22LV10CQZ-30PU) |
| | socket | 1 | [Amphenol DILB24P-224TLF](https://www.mouser.com/ProductDetail/649-DILB24P-224TLF) |
| Case | Hammond 1593K | 1 | [Hammond 1593KTBU](https://www.mouser.com/ProductDetail/546-1593KTBU) |
| Screws | Hammond self-tapping | (4 of) | [Hammond 1593ATS](https://www.mouser.com/ProductDetail/546-1593ATS100) |
