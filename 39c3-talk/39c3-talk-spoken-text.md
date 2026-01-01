# Script for 39C3 talk: 'Pwn2Roll: Who Needs a 595€ Remote When You Have wheelchair.py?'

## Table of Contents
- [PROLOGUE: "The Discovery"](#prologue-the-discovery-5-min)
- [ACT 1, Scene 1: The Paywall Catalog](#act-1-scene-1-the-paywall-catalog-5-min)
- [ACT 1, Scene 2: "Cyber Security Key"](#act-1-scene-2-cyber-security-key-5-min)
- [ACT 2, Scene 3: The Code](#act-2-scene-3-the-code-6-min)
- [ACT 2, Scene 4: The Protocol](#act-2-scene-4-the-protocol-5-min)
- [ACT 2, Scene 5: wheelchair.py](#act-2-scene-5-wheelchairpy-7-min)
- [ACT 3, Scene 6: The Bigger Picture](#act-3-scene-6-the-bigger-picture-5-min)
- [FINALE: The Liberation](#finale-the-liberation-8-min)

This document is more or less the whole content I intended to be included in the talk - due to last-minute mess-ups with my slide set only parts were part of the talk. So this documents acts as a reference, also to explain the code and it's history in the [Github code repository](https://github.com/roll2own/m5squared). 

## PROLOGUE: "The Discovery" (~5 min)

I use a wheelchair. I had it for years. Then I got an Alber e-motion M25 DuoDrive - electric motors that replace the wheels. You push, they amplify. Hills become flat. The DuoDrive upgrade adds self-driving - set a speed, hands off, it goes. Good hardware, actually. I appreciate it.

I even paid €3000 out of pocket for the self-driving upgrade. Spoon theory - my energy levels vary.

Why out of pocket? The insurance took six months just to approve the basic drive. I wasn't waiting another six months for a 'maybe' on the upgrade.

The device came with two remotes. €500 for the basic one. €980 for the one with a knob. Plus tax. Both use Bluetooth.

Bluetooth.

We all know how that usually goes.

I also found out there's an app. Free download, so I downloaded it. Opened it up to see what it could do.

Some things are free: battery level, switching between drive profiles. The basics.

But then: in-app purchases. €99.99 for speed unlock. €99.99 for 'remote control.' €99.99 for cruise mode. €39.99 for parking mode. They call it the 'Mobility Plus Package.' There's a 30-day trial for everything.

Assistance level 2 (more motor power for outdoor use) is locked behind the remote feature. Level 1 is indoor mode. Want to go outdoors, esp. uphill? Pay up.

And there's a password-locked 'dealer mode.' Because some settings are apparently only for the royal high society.

Now. The password.

Let's say it wasn't hidden very well.

But if you prefer official sources: it's in the manual. 'Information for Therapists and DME Dealers.' Page 8. Dutch version. Or French. Or Portuguese. Take your pick, I also posted it on Mastodon last year as it's public knowledge.

The English and German versions? Redacted. But they forgot about the translations.

So I unlocked dealer mode. And there it was.

An assist mode toggle. The same function as the €99.99 remote feature.

Working.

But I hadn't bought that feature.

Why does this work?

So paywall is... optional?

Now I *had* to know. How does this thing actually work? What's real? What's theater?

That was fall 2024. I've been having a lot of fun since then.

Welcome to Pwn2Roll.

---

## ACT 1, Scene 1: The Paywall Catalog (~5 min)

Before we get into how it works - let me show you what they think it's worth.

Five buttons on the remote. €500. That's €100 per button.

Oh, and this isn't new. The Twion which was discontinued in 2022 had also a Mobility Plus Package for €390. Since 2014.

They're pioneers. Of wheelchair microtransactions.

The online catalog shows NET prices. 'zzgl. MwSt' - plus VAT.

That €8,000 DuoDrive? €9,500 at checkout. Dealers reclaim VAT, insurance pays without - but when you're paying yourself, even only specific add-ons or features? Full price - including the VAT.

The catalog seems less honest than Apple.

I paid €3000 for the DuoDrive upgrade. What did I actually get for that price?

Same motors. Same controller board. Same everything.

The only difference? A configurable flag in firmware. And an even pricier remote than the €500 remote.

`IsDuoDrive = true`

Three thousand euros. For a `true`.

But wait! It gets better.

[pointing at slides]

Left side: the €99.99 'Remote Control' in-app purchase.

Right side: the dealer mode toggle. Same function. Works without paying.

The password was on page 8. Non-English and non-German therapist manuals.

They're charging €99.99 for permission to use a toggle that already works.

They call it 'Profimodus.' Professional mode, Dealer-only settings.

I'd rather call it 'Peasantmodus was too honest.'

The M25 drive is not the only product including DRM payment features: including the Twion there's five products in total.

---

## ACT 1, Scene 2: "Cyber Security Key" (~5 min)

The drive uses AES-128 encryption key for all Bluetooth communication - smartphone, both remotes and the maintenance application for dealers. The AES key? It's encoded and printed on a QR code on a sticker, onto each wheel's hub.

On the outside of the wheelchair.

The call it - and I'm not making this up - 'CYBER SECURITY KEY.'

I want to meet the person who named this. I have questions. And a thesaurus.

This is like tattooing your SSH private key on your forehead and calling it 'biometric security.'

So how does that QR code become an encryption key?

It's 22 characters. Custom 64-character alphabet. Each character maps to 6 bits.

22 times 6 equals 132 bits. But AES 128 needs 128 bits, so drop the first 4. You get 128 bits.

That's your AES-128 key. The entire security of the device.

Anyone who scans that QR code from the hub can decrypt - and encrypt - all your traffic.

That's not a vulnerability. That's a design choice.

Here's the beautiful part: the crypto looks actually good. AES-128-CBC. Textbook implementation. Someone knew what they were doing.

Which makes printing the key on a sticker even funnier.

Apparently crypto team and the key distribution team clearly never met (if they exist).

Back in 2020, SweynTooth was disclosed. A bundle of Bluetooth vulnerabilities, also affecting a lot of medical devices which i.e. can cause Denial of Service.

The manufacturer had to file with the FDA. Their mitigation strategy was sort of creative.

[reading text from slide]

- 'The device would only stop functioning if actively attacked.'

That's the vulnerability. You just described the problem and called it a mitigation.

- 'An attacker would need to be within Bluetooth range.'

Security by not being in physical range. Revolutionary security thinking.

Three: 'Bluetooth would need to be enabled for the attack to work.'

Bluetooth cannot be disabled on the drive, you'd need to desolder the Bluetooth module.

Their filing isn't a mitigation. This is describing how Bluetooth works. And it seems the BLuetooth module's firmware was never updated to a firmware including the real fix, it's still broadcasting the 1.00 firmware version from 2016.

Their security posture apparently is 'Have you tried not being attacked?' Bold move.

---

## ACT 2, Scene 3: The Code (~6 min)

So the paywall is theater. The security is a sticker. Now I wanted to understand everything.

The good thing is: I actually love reverse engineering. Wireless protocols. Encryption. Security quirks. This was going to be fun.

I started with the Android app. Standard stuff - jadx is your friend and looked at what it's doing.

The app has Google Play Billing on Android, Apple's StoreKit on iOS. Same paywalls, different payment processors. The interesting part isn't how they charge - it's what they're charging for.

And then I found this:

One method call. `isSpeedEnabled()`. Returns true? 8.5 km/h. Returns false? 6 km/h.

The price difference? €99.99.

€99.99 per `true`.

Let me tell you what that boolean actually costs in a real-world example:

8.5 km/h. 400 meters to the bus stop. That's 2 minutes 50 seconds.

6 km/h. Same distance. 4 minutes.

The bus leaves when it leaves.

Miss it by 70 seconds because you didn't pay €99.99 for a bit flip.

That's not a hypothetical. That's a Tuesday.

Oh, and here's the fun part about the speed increase:

In Germany, wheelchairs above 6 km/h need registration. Which means insurance, paperwork, etc.

So the €99.99 speed unlock? It doesn't just unlock speed.

It unlocks your ability to break the law.

€99.99 for a crime-as-a-service subscription.

The app has a 30-day trial for all premium features. It the whole so-called 'Mobility Plus Package'.

And here's how it works: when you activate the trial, it sets a flag on the hardware-side of the drive. The drive counts down 30 days. Fair enough. And then the trial stays active for infinite time.

But here's the thing:

The drive itself? It doesn't verify purchases. At all. It just accepts whatever speed parameters the app sends.

The DRM is entirely in the phone app. The app checks: 'did they pay and is the trial active?' If yes, show the button. If no, hide the button.

The purchase doesn't unlock any functionality. It unlocks the GUI parts for accessing the functionality of the drive. The button was always there, they just hide it if you don't pay them.

The App Store and Play Store reviews speak for themselves, but not the reviews themselves, it's the manufacturer's answers:

[reading the German part first, then the English translation]

'Der In-App Kauf ist an Dein Googlekonto gebunden.'

"The in-app purchase is bound to your Google account.'

Don't take my word for it. Here's the manufacturer saying it themselves:

'The in-app purchase is bound to your Google account.'

They said it themselves. The drive doesn't know. The drive doesn't care. The paywall lives in the app.

Which means anyone who paid for the speed feature can set your drive to 8.5 km/h. Without you paying. The drive doesn't check whose app store profile sent the command.

The €8,000 drive trusts whatever it receives. So when I use Python to send the high speed parameters directly? I'm not bypassing any protection. There's nothing to bypass.

I'm just not using the app to speak to the wheelchair's API. That's it. That's the 'circumvention.'

This isn't DRM. This is a suggestion.

And the security theater doesn't stop at the paywall.

Let me show you their secrets management.

The dealer mode password? Company initials, plus the model number, plus some punctuation. Creative.

The service mode password - the one that unlocks full access when the maintenance application is used? Error memory, factory reset, firmware upload, even changing the encryption key?

If you learned to count as a child: one, two, three...
...eight digits. You get the idea.

The USB maintenance key? Company name, underscore, product name. Padded to 16 bytes since it's AES 128. Same key in every drive. Hardcoded. Also sitting in the firmware binaries, if you know where to look.

And the config exports from the dealer software? Base64-encoded XML. Including the AES key of the drive and the remotes. In plaintext. In a world-readable folder.

Security through 'we hope you don't try.'

---

## ACT 2, Scene 4: The Protocol (~5 min)

So let's get to the proprietary protocol.

'Proprietary.' They keep using that word. I don't think it means what they think it means.

The packet structure is:

`0xEF` magic byte. Length in byte. Encrypted payload. CRC.

That's not proprietary. That's a CS homework assignment.

Here's encrypted traffic from the wheelchair. Looks like noise, except for the magic byte, length, etc.

[reference the slides]

Now the same traffic, decrypted with key derived from the QR code.

Suddenly it's got more meaning. A per-message incrementing counter. Message direction. Service ID's. Parameter ID's. Configuration values. Telemetry data.

The protocol has about 14 services: battery management, drive profiles. statistics, RTC, etc.

Each service has parameter ID's. Read battery level? Service 2, Parameter 1. Change assist level? Different service, different parameter.

It's actually well-organized. The reverse engineering was mostly grunt work:

Dumping USB traffic of the maintenance application, dumping Bluetooth traffic from the Smartphone app. Weeks of Wireshark at night. Staring at loads of hexadecimals. Writing helper scripts that turn the hexadecimals into their actual meaning. Perfect way to spend weekends.

So what does the €500 ECS remote actually send?

It's five functions:

1. Power On / Standby
2. Assist Level 1 (short press) - indoor mode
3. Assist Level 2 (short press) - outdoor mode
4. Learning Mode (long press) - reduced speed for getting to know how to handle the drive
5. Rollback Delay toggle - hill-holder, prevents rolling backwards when stopping

That's it. Five parameter toggles. No algorithms. No math. Just 'set this value to 1' or 'set this value to 2.' Plus reading out the configured parameters and some more information, e.g. the battery level of the drive.

€500 for five Bluetooth toggle switches. €100 per toggle.

I've seen more features in a TV remote. That came free with the TV.

The €980 'premium' remote with the knob?

There's no display. Same Texas Instruments Bluetooth module.

Plus...
A big turning wheel. An 18650 battery plus USB charging PCB. Power button. Assist mode switching - but not all modes without the learning mode option. No auto-hold toggle.

It does LESS than the €500 remote. But here's the thing:

Cruise mode - that's the self-driving feature. What makes DuoDrive the DuoDrive. Set a speed, hands off, it goes.

Parking mode - remote control via virtual joystick. Move your wheelchair without sitting in it. Park it in a corner, summon it across the room, or send your wheelchair to get some Tschunk at congress.

You CAN control both from the app.

But cruise mode from a touchscreen means adjusting your speed by tapping glass. While moving. Emergency stop? Also on the phone. Which you need to hold somehow while you're moving.

The knob is a physical dial. Turn it forward, you speed up. Push the dial for emergency stop. The eyes stay on the road.

So the €980 knob is genuinely useful - it's the only **safe** way to use cruise mode. The app alternative is, let's say, 'exciting'.

But €480 for 'your wheelchair won't drive you into the railway tracks'? That's not a feature. That's a ransom.

Apparently they're not selling mobility aids, maybe they're selling audacity.

---

## ACT 2, Scene 5: wheelchair.py (~7 min)

Once you understand the protocol, the rest is just Python.

To give you an overview:

- `m25_qr_to_key.py`: turns that QR code into an AES key

- `m25_decrypt.py` and `m25_encrypt.py`: packet cryptography handling

- `m25_analyzer.py`: makes the decrypted packets more human-readable

- `m25_protocol.py`, `m25_protocol_data.py`: the protocol specs

- `m25_ecs.py`: all the ECS remote functions plus speed unlock

- `m25_test_parking_forward.py`: just to demonstrate the parking feature

I named the collection wheelchair.py. as 'vengeance.py' was too on the nose. Barely.

[starting live demo part here]

Let's see what we built.

This is what the €500 remote does when switching assist modes.

The Python script can do the same.

This is what €99.99 unlocks in the app:

8.5 km/h. Catch the bus. No boolean required. The Python script can do the same.

And this is maintenance access.

Everything the dealer tools can see and write.

No remote. No app. No €99.99. Just some basic Python talking to the hardware and reading and intepreting what the hardware sends.

Now the other remote control:

Here's what €980 buys you. A potentiometer. A dial. That's the knob.

Maybe €30-40 in parts, if you're feeling generous.

€480 more than the €500 remote.

€480 for a knob.

The €500 remote? Replaced.

The €980 knob remote? Replaced.

The €99.99 speed unlock? Replaced.

The €99.99 remote app feature? Replaced.

The €39.99 parking feature? Replaced.

The maintenance access which is restricted to dealers? Replaced.

Total savings: about €1,800.

Time spent: about 200 hours.

Hourly rate: terrible.

Fun: absolutely worth it.

---

## ACT 3, Scene 6: The Bigger Picture (~5 min)

I've been having fun with €99.99 booleans and €480 knobs.

But there's a reason I'm telling you this at CCC instead of doing quietly for myself.

Here's the German twist.

I don't own this wheelchair. The Krankenkasse - German health insurance - does. Forever.

Even the €3,000 I paid out of pocket for the upgrade? Still their property.

The only thing I actually own? The €99.99 app features.

The features that don't enforce anything.

I own permission slips for a paywall that doesn't lock.

What happens when they discontinue the app?

When Android 19 breaks compatibility?

Your €8,000 wheelchair - remember, plus VAT - becomes a very expensive planter.

Unless someone documented the protocol.

Unless someone built tools.

That's not hypothetical. Companies abandon apps. Devices brick. It happens.

This pattern isn't unique.

John Deere won't let you fix tractors.

HP won't let you use third-party ink.

Car manufacturers sell heated seats as a subscription.

And wheelchair drives have in-app purchases.

What's next? Pacemakers with premium heartbeats?

The canary in the coal mine. And the canary has microtransactions.

---

## FINALE: The Liberation (~8 min)

So. What next?

I'd love to release everything today.

So I am.

Everything. Right now. It's live.

The full protocol documentation.

The drive's and remote's architectures. The packet structure. Service IDs, Parameter IDs. Command references.

All the stuff the manufacturer never published.

The Python toolchain:
- `m25_qr_to_key.py`
- `m25_bluetooth.py`
- `m25_decrypt.py` / `m25_encrypt.py`
- `m25_protocol.py`
- `m25_analyzer.py`
- `m25_ecs.py`
- `m25_parking.py`

The code is free. Go poke at things, but don't scan QR codes from people's drives and mess with them.

Side note, I'm not publishing everything I found:

Certain security findings.

Those need a proper disclosure process. The manufacturer will respond eventually. Or they won't. But I'll have documentation either way.

If you're a security researcher who wants details - find me. We'll talk.

If you're the manufacturer - hi. My email works. Does yours?

Assistive technology isn't just a product you use. It replaces or augments parts of your body - so it's an extension of yourself.

When a company puts a paywall on my mobility, they're not locking a feature. They're locking part of me.

I think we all know that understanding the devices we depend on isn't optional. It's necessary.

The alternative is trusting money-making corporations with our bodies. And hoping they don't discontinue the support for all of the hard- and software or enshittify it otherwise.

If you have an M25, you can now control it without the paywalls.

Have fun and void your warranty.

If you have a different medocal device - wheelchair, prosthetic, CPAP, whatever - apply the methodology.

Capture the traffic. Read the code. Document the protocol. Build the tools. Stuff like this needs more public attention.

The protocol of the M25 drive is documented. The commands are known.

Someone could build an open-source remote for €50 in parts. ESP32, a potentiometer, a button. Done.

If you do, let me know. I'll test and link it.

Support right to repair legislation.

Ask why insurances pay €500 for five Bluetooth commands.

Ask why we don't own the devices we depend on.

The remotes and in-app-purchsed features represent artificial scarcity.

The specs I documented and the tools I built represent autonomy.

The choice is yours.

Catch the bus.

Don't pay €99.99 for a boolean.

Thank you.
