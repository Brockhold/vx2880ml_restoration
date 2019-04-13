#### My efforts to restore a mysteriously dead Viewsonic monitor.

# The story so far:
With the monitor open, I probed out the voltage supplies and found no obvious issues.
My Viewsonic VX2880ML suddenly stopped working, with curious symptoms. When power was applied, it would turn on the backlight and the power LED, but was otherwise unresponsive. Interestingly, the host computer would be able to negotiate a connection, and correctly read its properties. I bought my monitor refurbished, which turns out to mean the warentee (normally pretty good for Viewsonic products) is just 90 days. Tech support was polite but unhelpful. With nothing to lose but time and patience, I decided to dig deeper.

After searching the web, I found exactly one other person discussing this problem on two different forums.
["Armogun" posted on the russian professional electronics forum remont-Aud, looking for advice,](https://remont-aud.net/forum/23-69117-1)
and they [also posted on Espec and uploaded some relevant files!](http://monitor.espec.ws/section33/topic285657p20.html)
Their monitor problems sounded (thanks, Google & Bing translation) just like mine, and Armogun determined that the contents of one of its flash memories was corrupt. They reported that after flashing, their monitor retuned to life. So, worth a shot.

I popped the monitor open and located the main controller (STDP9320). According to its data sheet, the controller loads its firmware from an adjacent SPI-connected flash memory. In this case, those are components *U17* and *MU7*, which are MX25L1606E. It turns out the controller has a neat feature: you can upload firmware to it via DisplayPort via "EZ-Display UP." This feature is not documented anywhere I can find, and the protocol is not publicly available. I'm suspicious that it's responsible for the firmware becoming inoperable. I'd love to see a program that can speak this protocol to a monitor.

To dump the contents of the flash memory, I built a programmer using an Arduino Duemilanove with a Sparkfun 5v->3v level translator, plus a custom PCB to provide a surface mount footprint to solder the component to. The PCB is honestly overkill, but I had some board space left over. I've added a photo of my setup, it's quite simple. I had to transfer the flash memories off the motherboard due to the design of the board; I was hoping to be able to program them in-place, but was unable to safely sink enough current to pull the chip select pin low. With a little more advanced programmer design I imagine it would be possible, but I didn't want to risk damaging anything on the motherboard.
Using [Flashrom](https://www.flashrom.org/Flashrom) on my Ubuntu desktop and [frser-duino](https://github.com/urjaman/frser-duino/) on the arduino, I was able to dump the firmware.

I compared the dump from my monitor with the two dumps Armogun provided, one was known bad, and one was known good. Mine is distinct, but more similar to the corrupt dump than the known-good dump.
    `diff -y --suppress-common-lines  <(xxd my_MU7.bin ) <(xxd good_MU7.BIN ) | colordiff`
Browsing the `strings` in the flash dumps, it's evident that there's some sections missing in the corrupt versions. They also have different (earlier) version numbers, which is interesting. I've included the string dumps here.

After flashing the good MU7 contents I transferred the component back to the monitor motherboard, and... *it works!*

Big thanks to Armogun and Mark7, for providing the flash memory dumps, and reversing enough of the motherboard to make my job easy. Thanks also to Urja Rannikko for frser-duino, and Stefan Tauner for MX25L1606E support in Flashrom.
