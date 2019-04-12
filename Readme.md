My efforts to restore a mysteriously dead Viewsonic monitor.

The story so far:
My Viewsonic vx2880ml suddenly stopped working, with curious symptoms. When power was applied, it would turn on the backlight and the power LED, but was otherwise unresponsive.
Interestingly, the host computer would be able to negotiate a connection on HDMI, and correctly read its properties.
With the monitor open, I probed out the voltage supplies and found no obvious issues.

After searching the web, I found exactly one other person discussing this problem, on two different forums.
"Armogun" posted on the russian professional electronics forum remont-Aud, looking for advice: https://remont-aud.net/forum/23-69117-1
They also posted on Espec: http://monitor.espec.ws/section33/topic285657p20.html
Their monitor problems sounded (thanks, Google & Bing translation) just like mine, and Armogun determined that the contents of the MU7 flash memory (MX25L1606E) were corrupt.

I popped the monitor open and located the flash memory, as well as the image scalar (STDP9320) to which it corresponds, which has a neat feature: you can upload firmware to it via DisplayPort. This feature is not documented anywhere I can find, and the protocol is not publicly available.
To dump the contents of the flash memory, I built a programmer using an Arduino Duemilanove with a 5v->3v level translator, plus a custom PCB to provide a surface mount footprint to solder the component to.
Using Flashrom https://www.flashrom.org/Flashrom on my Ubuntu desktop and frser-duino on the programmer https://github.com/urjaman/frser-duino/ I was able to dump the firmware.
I compared the dump from my monitor with the two dumps Armogun provided, one was known bad, and one was known good. Mine is distinct, but more similar to the corrupt dump than the known-good dump.
`diff -y --suppress-common-lines  <(xxd my_MU7.bin ) <(xxd good_MU7.BIN ) | colordiff`
After flashing the good MU7 contents I transferred the component back to the monitor motherboard, and... it works!

Big thanks to Armogun and Mark7, for providing the flash memory dumps, and reversing enough of the motherboard to make my job easy.
