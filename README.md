The Problem
Many commercial AV environments run Q-SYS for audio DSP and control alongside Crestron DM-NVX for video-over-IP distribution. The traditional approach requires a dedicated Crestron processor (CP4-N, RMC4, etc.) just to manage the NVX endpoints — adding cost, complexity, and a second programming language to maintain.

Consolidating control into Q-SYS reduces hardware failure points, simplifies programming, and provides operators with a single unified touch panel interface.

Integration Approaches
1. REST API (Manual Lua Scripting)
NVX endpoints expose a REST API over HTTPS. From Q-SYS, you can use the HttpClient library in a Text Controller or Control Script to issue routing commands.

Example: A basic Lua script for routing a video stream

lua

-- Manual NVX Video Routing Example
-- Note: This requires writing your own polling, auth handling, and UI elements.
nvxAddress = "192.168.1.50"
streamUrl = "rtsp://192.168.1.40:554/live.sdp"
function RouteVideo()
  local headers = {
    ["Content-Type"] = "application/json",
    ["Accept"] = "application/json"
  }
  
  local payload = {
    ["Device"] = {
      ["Routing"] = {
        ["Video"] = {
          ["ReceiveMethod"] = "Stream",
          ["MulticastAddress"] = streamUrl
        }
      }
    }
  }
  
  HttpClient.Upload({
    Url = "https://" .. nvxAddress .. "/Device/Routing",
    Method = "POST",
    Headers = headers,
    Data = rapidjson.encode(payload),
    EventHandler = function(response, code, headers, errorMsg)
      if code == 200 then
        print("Route successful")
      else
        print("Manual route failed: " .. (errorMsg or "Unknown error"))
      end
    end
  })
end
-- UI Trigger
Controls.RouteButton.EventHandler = function(ctl)
  if ctl.Boolean then
    RouteVideo()
  end
end
The catch with manual scripting: While the basic routing call above looks simple, a production-ready script requires authentication handling (session tokens), HTTPS certificate validation bypasses, error handling, heartbeat polling for offline detection, and building custom UI elements. Building this robustly takes 15–20 hours of development and testing time.

2. Pre-Built Q-SYS Plugin (Recommended)
Instead of maintaining thousands of lines of custom Lua and dealing with Crestron's API quirks, you can use a purpose-built plugin that handles the protocol layer, authentication, and error recovery natively.

CoreForge Labs — Crestron NVX Control Plugin

This production-ready Q-SYS plugin drops directly into your design and handles:

Video routing — switch sources across NVX transmitters and receivers natively
Audio routing — independent audio stream selection (breakaway routing)
USB routing — USB-over-IP switching for BYOD and KVM workflows
Device status — real-time feedback for connection state, firmware version, and stream health
No Crestron processor required — communicates directly with NVX endpoints
One-time per-Core licensing — no subscriptions, run unlimited instances on a single Q-SYS Core
Supported NVX Hardware
Model	TX/RX	Video	Audio	USB
DM-NVX-350 / 350C	TX/RX	4K60 4:2:0	✅	✅
DM-NVX-351 / 351C	TX/RX	4K60 4:4:4	✅	✅
DM-NVX-360 / 360C	TX	4K60 4:4:4 HDR	✅	✅
DM-NVX-363 / 363C	RX	4K60 4:4:4 HDR	✅	✅
DM-NVX-E30	Encoder	4K60 4:4:4	✅	❌
DM-NVX-D30	Decoder	4K60 4:4:4	✅	❌
FAQ
Q: Do I need a Crestron processor (CP4/RMC4) to use this? A: No. The plugin communicates directly with the NVX endpoints over their REST API.

Q: How is it licensed? A: One-time purchase, per Q-SYS Core. Run unlimited instances of the plugin on that Core. No subscriptions.

Q: Can I test it before buying? A: Yes. The plugin runs in full emulation mode in Q-SYS Designer at no cost so you can verify the UI and control pins before deployment.

Resources & Links
Download the NVX Plugin - https://coreforgelabs.org/products/crestron-nvx-control-plugin
 — CoreForge Labs
View All Q-SYS Plugins - https://coreforgelabs.org/collections/all
 — Full catalog
Per-Core Licensing Explained
