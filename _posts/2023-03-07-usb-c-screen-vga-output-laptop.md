---
layout: post
title: "Connecting a (portable) USB-C screen to an old notebook"
date: 2023-03-07
categories: Connecting USB-C screen VGA old notebook Thinkpad X201
comments: true
---

To connect a USB-C screen to an old notebook, such as the Thinkpad X201, there are a few options:

# DisplayLink over USB-3

Install the [DisplayLink drivers](https://www.synaptics.com/products/displaylink-graphics/downloads) to use your USB-3 port.
If your laptop has none, then likely it has a PCI-Express (PCMCIA) card slot in which a [(54mm Slot) USB 3.0](https://www.amazon.de/CSL-PCMCIA-Express-Windows-Notebook/dp/B008BHN6MI) [ExpressCard](https://www.aliexpress.com/item/4000281886119.html) can be inserted.
The USB 3.0 slot emits at least 5 Watt, which might be almost sufficient for, say, the [Asus MB16AC](https://www.asus.com/us/product-compare?ProductID=12352%2C15369%2C8886%2C18041&LevelId=Displays-Desktops-Monitors), needing between 5 and 8 Watt.
Since power supply is likely somewhat low, a [dual power](https://www.amazon.de/dp/B07VVX257H) [Y-cable](https://www.amazon.de/dp/B07VPNSMN8) with two USB-3 and one USB-C male connectors such as [Vention's](https://www.aliexpress.com/item/1005002704214182.html) provides additional power.

# HDMI to USB-C Converters

For possible HDMI (male) to USB-C (female) converters, that is, from an  HDMI port (of the Laptop) to a USB-C port (of a portable Monitor such as the [Asus ZenScreen](https://www.notebookcheck.com/Test-Asus-ZenScreen-MB16AC-15-6-FHD-IPS-Monitor.251642.0.html)) see first [Dan's blog post](https://dancharblog.wordpress.com/2020/05/10/bi-directional-usbc-dp-cables/#for-portable-usb-c-touch-screens-and-vr).
There are also now the more affordable [Fairikabe ](https://www.amazon.com/fairikabe-Converter-Thunberbolt-Compatible-Portable/dp/B0B5XBYQSM)
and [Elebase ](https://www.amazon.com/dp/B08VDT3YGK) (which is also sold as [Basesailor ](https://www.amazon.de/Micro-USB-HDMI-Eingang-3-1-Ausgang-konverter-Thunderbolt/dp/B09LGVNXPK)) which perform fairly [comparably ](https://r.nf/r/nreal/comments/ziredh/has_anyone_tried_this_fairikabe_hdmi_to_usbc/).

# VGA to HDMI Converters

If the laptop is so ancient that is only sports a VGA port (such as the Lenovo Thinkpad X201), there are convenient VGA (male) to HDMI (female) converters such as that from [HAMA](https://at.hama.com/00200342/hama-video-adapter-vga+usb-stecker-hdmi-buchse-full-hd-1080p)
and the [ICZI VGA to HDMI Converter](https://aliexpress.com/item/32735736724.html) that occupy a single additional USB port for power supply and sound and turn the laptop's VGA into a fully capable HDMI port.
