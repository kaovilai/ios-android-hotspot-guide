# iOS and Android Hotspot Guide

Practical, privacy-conscious patterns for joining, sharing, and triggering personal hotspots across Apple and Android devices.

## What this guide covers

- Joining an iPhone or iPad hotspot from another Apple device.
- Joining an iPhone or iPad hotspot from an Android device.
- Joining an Android hotspot from an iPhone or iPad.
- Turning on an Android hotspot from a Mac, iPhone, or iPad with a local bridge.
- Turning a hotspot on automatically with Shortcuts, Android automation apps, or local hardware triggers.
- Avoiding common cross-platform limits and accidental data use.

## Core rules

1. **Apple Instant Hotspot only works inside Apple's ecosystem.** It requires the same Apple Account or the same Family Sharing group. An Android device cannot use it.
2. **Android and iOS can join each other as ordinary Wi-Fi clients.** That path always needs the hotspot password unless the platform provides its own trusted-device shortcut.
3. **A hotspot must be on before a client can join it.** Cross-platform clients cannot wake an iPhone or Android hotspot by merely scanning for the network.
4. **Explicit automations belong on the provider.** If you use Shortcuts or a vendor automation app, put the automation on the device providing the hotspot. Apple Instant Hotspot is the client-side exception.

## Quick matrix

| Client | Hotspot provider | Password-free? | Recommended path |
| --- | --- | ---: | --- |
| iPhone or iPad | iPhone or iPad on the same Apple Account | Yes | Instant Hotspot |
| iPhone or iPad | iPhone or iPad in the same Family Sharing group | Yes | Instant Hotspot with Family Sharing |
| Android | iPhone or iPad | No | Join as a normal Wi-Fi network |
| iPhone or iPad | Android | No | Join as a normal Wi-Fi network |
| Android | Android | Usually no | Normal Wi-Fi; some vendors add trusted-device shortcuts |
| Mac, iPhone, or iPad | Android | No | Custom BLE, webhook, or home-automation bridge |

## Prerequisites

### On the hotspot provider

- A mobile data plan that supports tethering.
- Mobile data enabled.
- Hotspot or tethering available in Settings.
- A strong hotspot password.
- A stable device name so clients can recognize the network later.

### On the client

- Wi-Fi enabled.
- Enough battery to stay connected.
- The hotspot password, unless using Apple Instant Hotspot.

## Apple to Apple

### Same Apple Account

Use this when both devices belong to you.

1. On the hotspot provider, open **Settings > Personal Hotspot**.
2. Confirm Personal Hotspot is available.
3. On the client, open **Settings > Wi-Fi**.
4. Tap the provider's device name.
5. If both devices are signed in to the same Apple Account, no hotspot password should be required.

### Family Sharing

Use this when the devices belong to different people in one Family Sharing group.

On the hotspot provider:

1. Open **Settings > Personal Hotspot > Family Sharing**.
2. Turn on Family Sharing.
3. Tap the person's name.
4. Choose **Automatic** or **Ask for Approval**.

On the client:

1. Open **Settings > Wi-Fi > Auto-Join Hotspot**.
2. Choose **Automatic** or **Ask to Join**.

With Instant Hotspot, the provider does not need to leave **Allow Others to Join** enabled for trusted Apple devices.

## Apple hotspot to Android

1. On the iPhone or iPad, open **Settings > Personal Hotspot**.
2. Turn on **Allow Others to Join**.
3. On Android, open **Settings > Network & Internet > Wi-Fi**.
4. Tap the Apple device's name.
5. Enter the hotspot password.
6. Let Android save the network.

If the Android device cannot see the hotspot:

- Keep the Apple device on the Personal Hotspot screen during the first attempt.
- Turn on **Maximize Compatibility** if available.
- Restart both devices.
- Confirm the hotspot password uses supported characters.

## Android hotspot to Apple

1. On Android, open **Settings > Network & Internet > Hotspot & tethering**.
2. Turn on the Wi-Fi hotspot.
3. Set or verify the network name and password.
4. On the iPhone or iPad, open **Settings > Wi-Fi**.
5. Tap the Android hotspot.
6. Enter the password.
7. Let the Apple device save the network.

If the Apple device cannot see the hotspot:

- Keep the Android hotspot screen open during the first attempt.
- Temporarily disable any hidden-network or band-specific setting.
- Restart both devices.

## Android to Android

1. On the provider, enable the Wi-Fi hotspot.
2. On the client, open Wi-Fi settings.
3. Tap the hotspot name.
4. Enter the password.
5. Let the client save the network.

Some Android vendors offer trusted-device or same-account hotspot features. Those features normally work only between devices from the same vendor or signed in to the same account.

## Triggering an Android hotspot from Mac or iOS

Apple devices cannot use Apple Instant Hotspot to wake an Android hotspot. A Mac, iPhone, or iPad can still trigger one if you add a bridge on the Android side. The bridge is a separate protocol, not an Apple Continuity feature.

### Mac-to-Android BLE bridge

This is the closest practical equivalent to Instant Hotspot:

1. Install an Android app that exposes a Bluetooth LE GATT hotspot controller.
2. Grant it access through Shizuku or another privileged helper.
3. Pair the Mac and Android device once.
4. Use a Mac menu-bar app, script, or shortcut to send the hotspot-on command.
5. Let the Mac client join the Android hotspot with the saved password.

Open-source examples include [Untether](https://github.com/ImOstrovskiy/untether) and [EasySpot](https://github.com/EasySpotApp/EasySpot). They use Android 12+, BLE advertising, and Shizuku to toggle tethering without root. Untether also signs commands over the BLE link and can join the hotspot from the Mac side.

Trade-offs:

- It works without SMS, cellular data, or a shared Wi-Fi network.
- It needs a persistent foreground service or background service on Android.
- It may need battery-optimization exemptions.
- It is device- and Android-version dependent.
- It does not make Android participate in Apple Family Sharing.

### iOS-to-Android local bridge

An iPhone or iPad can trigger an Android hotspot through a local bridge if both devices are reachable:

1. Run an automation server or webhook receiver on the Android device.
2. Expose an authenticated local HTTP endpoint or a Home Assistant entity.
3. Create an iOS shortcut that calls that endpoint when you need the hotspot.
4. Have the Android side toggle the hotspot with Tasker, MacroDroid, or a vendor automation app.

Example pattern:

```text
iOS Shortcut
  -> HTTPS request to Home Assistant or local webhook
  -> Android automation app
  -> Android hotspot toggles on
```

Use this only on a trusted network, or through a VPN or reverse proxy with authentication. Do not expose an unauthenticated hotspot-toggle endpoint to the public internet.

An iPhone or iPad can also run a custom BLE GATT client, but Apple's built-in Shortcuts app does not provide raw BLE GATT control. That path requires a custom iOS app or a bridge device.

### Push or notification bridge

When a local network bridge is not available:

1. Send a push notification, webhook, or message from the Mac or iOS device.
2. Trigger a matching Android automation from that notification.
3. Turn the hotspot on only for trusted senders and unique codes.

This is more flexible than a purely local bridge, but it depends on network delivery and may introduce notification delays.

## Triggering an iPhone hotspot

### Native Apple trigger

Apple devices can activate a nearby family hotspot without opening Settings:

1. Put the provider and client in the same Apple Account or Family Sharing group.
2. Keep Wi-Fi and Bluetooth enabled on both devices.
3. Set the client's **Auto-Join Hotspot** option to **Automatic** or **Ask to Join**.

This is the only native proximity-based way for one device to wake another device's iPhone hotspot. A trusted message or email automation can also turn on the provider's hotspot after it arrives; see the fallback section below.

### Siri

On the hotspot provider:

1. Open Shortcuts.
2. Create a shortcut named **Hotspot On**.
3. Add the **Set Personal Hotspot** action.
4. Set it to **Turn On**.
5. Say: `Hey Siri, Hotspot On`.

Create a matching **Hotspot Off** shortcut with **Turn Off**.

### Leaving-home automation

On the hotspot provider:

1. Open Shortcuts > Automation.
2. Create a Leave automation.
3. Choose the home location.
4. Run **Set Personal Hotspot > Turn On**.

Add a reverse Arrive automation for the home location, or a Wi-Fi automation that runs when the provider joins the trusted home network, to turn the hotspot off.

### Arrival automation

Use a location trigger for a place where a hotspot is regularly needed:

1. Shortcuts > Automation > Arrive.
2. Choose the location.
3. Run **Set Personal Hotspot > Turn On**.

### CarPlay or car Bluetooth

For CarPlay:

1. Shortcuts > Automation > CarPlay.
2. Trigger when the provider connects to the car.
3. Run **Set Personal Hotspot > Turn On**.

Add a CarPlay disconnect automation to turn it off.

For a generic car Bluetooth connection, use the Bluetooth connect trigger to turn the hotspot on. Shortcuts' Bluetooth trigger does not provide a disconnect option, so turn the hotspot off with a separate Arrive, time-of-day, or low-battery automation.

### Focus

Create a Focus named **Hotspot**:

1. Shortcuts > Automation > Focus.
2. When the Focus turns on, run **Set Personal Hotspot > Turn On**.
3. When it turns off, run **Set Personal Hotspot > Turn Off**.

The provider can then enable the hotspot from Control Center, the Lock Screen, Siri, or an Apple Watch.

### NFC

Put an NFC sticker in a recurring location, such as a car console or bag:

1. Shortcuts > Automation > NFC.
2. Scan the sticker.
3. Run **Set Personal Hotspot > Turn On**.

This uses no network data and can be faster than opening Settings.

### Physical button

Use a Bluetooth button or smart-button accessory that can run a shortcut:

1. Pair the button to the provider.
2. Assign a press to the **Hotspot On** shortcut.

This is useful when the provider wants a one-press control without using voice.

### Push or email trigger

A third-party automation app such as Pushcut can show a notification that runs the shortcut. Shortcuts can also use an email automation with a specific sender and subject.

These options avoid SMS, but they still depend on network delivery and may require the provider to tap a notification.

### Android-to-iPhone Bluetooth trigger

An Android device can sometimes be paired with an iPhone, and an iPhone automation can run when a paired Bluetooth device connects. This is worth testing, but it is not reliable because iOS may not maintain a connection with a generic Android phone the way it does with audio accessories or smart buttons.

## Triggering an Android hotspot

### Quick Settings

Add the hotspot tile to Quick Settings so the provider can turn it on with one tap.

### Voice assistant

Ask the device's assistant to turn on the mobile hotspot. Use the exact wording supported by the device and assistant version.

### Vendor automation

Many Android devices include a routines, modes, or rules app. Common triggers include:

- Time of day.
- Leaving or arriving at a location.
- Connecting to a Bluetooth device.
- Disconnecting from a trusted Wi-Fi network.
- Battery level.

Use the available **Mobile hotspot** action if the device supports it.

### Third-party automation

Apps such as Tasker or MacroDroid can:

- Toggle the hotspot.
- Send a push notification.
- Respond to NFC tags.
- Run a Wi-Fi or Bluetooth trigger.

Android restricts some hotspot controls by version and vendor, so verify the exact action on the device before relying on it.

### iOS-to-Android Bluetooth trigger

If an iPhone or iPad can pair with the Android provider, an Android automation may run when that Bluetooth device connects. As with the reverse direction, this is experimental and depends on vendor support.

## Messaging fallback

If local triggers are unavailable, a provider can create a communication automation:

1. Use a unique code such as `HOTSPOT-ON-CODE`.
2. Trigger only on messages from a trusted contact.
3. Run the hotspot-on shortcut.

Use a separate code to turn the hotspot off. Do not use a common word such as `hotspot` because unrelated messages could trigger it.

## What does not work

- An Android device cannot use Apple Instant Hotspot.
- An Apple device cannot use vendor-specific trusted Android hotspot features unless the vendor supports it.
- A client cannot wake a hotspot simply by scanning for it.
- iOS does not expose the point when another device tries to join a hotspot before authentication.
- Shortcuts' Wi-Fi and Bluetooth setting triggers run when a network or device connects, not when it disconnects.
- Apple's contact-based Wi-Fi password sharing is for ordinary Wi-Fi networks, not Personal Hotspot.

## Safety and cost controls

- Keep the hotspot password long and unique.
- Turn off the hotspot when it is not needed.
- Use a timeout if the platform supports one.
- Add an automatic off switch for home Wi-Fi, bedtime, or low battery.
- Check whether the carrier charges separately for tethering.
- Monitor data use if the client may download updates or sync media.
- Avoid public QR-code generators for hotspot passwords.

## Recommended architecture

For a mixed Apple and Android setup:

1. Use Apple Instant Hotspot between trusted Apple devices.
2. Save each hotspot password once on cross-platform clients.
3. Put provider-side automations on the device that shares the hotspot.
4. Prefer local triggers such as Wi-Fi state, location, CarPlay, NFC, or a physical button.
5. Keep messaging or push triggers only as fallbacks.
6. Add automatic shutoff rules to protect battery and data.
