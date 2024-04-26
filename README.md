# RoomOS Air


RoomOS Air is a web based controller for Cisco collaboration devices for off-the-shelf devices such as iPad, laptops, smart phones
etc that cannot run native RoomOS code. It is mimicking the native RoomOS UI as closely as possible, for a consistent and familiar user experience.

The app only provides the user interface for controlling the video device, not video or audio for the meeting itself.

## Demo

Try it on 🕹 [️ cisco-ce.github.io/airpad/](https://cisco-ce.github.io/airpad/)


## Requirements

- Any device with a modern web browser (Chrome, Safari, Firefox, Edge, ...)
- Local user access to a Cisco device
- The controller device must be on the same network as the Cisco device

It does not matter whether the Cisco device is cloud registered or on-premise.

Supported devices: Any modern RoomOS device (Room series, Desk series, Board series).

## Support

The app is created as a last resort solution where customers cannot use standard Cisco controllers, and are not supported
through normal channels such as TAC.

## Who needs this app?

As the name suggests, RoomOS Air will only contain a small subset of all the features provided by the full Cisco controller UI.

For customers that need a different solution than the Cisco Navigator, because:

- They need a wireless controller
- They want to use off-the-shelf tablet (such as iPad) rather than Navigator
- They require a features that the Navigator does not support (e accesibility features such as screen reader or larger text)
- They need to control a device with a smart phone
- They need special branding or user experience that the Navigator doesn't allow
  (eg the user interface must match the user interface on tablets in other rooms)

## How does it work?

It is a pure web app / web page. As any web app, it needs to be hosted on a server for the browser to reach it, but the
web app itself talks directly to the Cisco device using web sockets and device APIs, and does **not** use any web services.

The app uses the public APIs (xAPI) to talk to the Cisco device. It stores the last used login locally, and re-connets
to this automatically every time the app is launched.

## How to use

- Make sure you have a local user (integrator and user role) on your device. Verify this by logging in to the device's web interface first.
  You may need to accept the self signed certificate of the device first
- On the device of your choosing, open the web browser and go to the URL where RoomOS Air is hosted
- Make sure the device is connected to the same network as your Cisco device
- Enter the IP address / hostname, user name and password to your Cisco device, and tap connect
- That's it. If that didn't work, see the *Why can't I connect* section below

## Connection

The app may loose connection to the Cisco devices in various situations. When this happens, the app will
indicate this clearly and try to reconnect automatically. This can happen:

- Whenever the iPad goes to sleep (auto-lock)
- When the iPad or Cisco device looses network connection
- When the Cisco device reboots (eg after software upgrade)
- If the browser wants you to re-confirm the certificate

There is also an option to disconnect manually from the device, in the device info menu in the top left corner of the home screen.
This will also delete any data stored on the tablet.

## Native-like app

The recommended way to use the app on a dedicated device is to load the web app in kiosk mode or Guided Access.

For personal devices, save the app to your home screen (typically Share > Save to home screen).
This will usually remove the URL bar etc from the app and make it feel like a native app.

If you want to use it for multiple Cisco devices, you can add the web app to your home screen multiple times, once for each device.
Remember to rename the app to the name of the device, then launch each app instance and log in to the device from there.


## What's up with the certificates?

Out of the box, Cisco devices provide self signed certificates. Most browsers are not happy about that and require you to
confirm that you in fact trust the certificate. The quickest way to do that is to visit the web interface of the device itself.

Look in the devices settings for the IP address and enter it in your browser's URL bar, eg like https://10.47.234.211/.
If you haven't already visited this page, the browser should now warn you about the certificate and ask if you trust it. Once that is
done, you can log in from the RoomOS Air app with the same credentials. You should then see the login page.
Note that you may need to repeat this process from time, depending on your browser.

To avoid this for long term use, you can create your own your own Certificate Authority and certificates, and install these on your controller
device and on the Cisco device. See more about this below.


## Suggested deployment

- iPad running the web app in kiosk mode
- No password on the iPad, and long go-to-sleep delay
- Custom certificates on controller device and Cisco device
- Table stand for the table, with charging, or daily charging routine


## Why can't I connect

Typical reasons for login to fail:

- The tablet is not on the same network as the Cisco device
- You haven't accepted the security certificate
- You are not using the correct username / password (this is the Cisco device's login, not your Webex login)

Visit the device's web log in (https://deviceip, make sure it is **https**) to accept the certificate,
and log in to verify your username / password.

If you are able to log in to the device interface, but the controller login still doesn't work, please make sure that the
following configuration is set on the device. You can check this in the Settings

```
xConfiguration NetworkServices Websocket: FollowHTTPService
NetworkServices HTTP Mode: HTTPS (or HTTP+HTTPS)
```

## Authentication / security

The device uses the browser's local storage to save the IP address, username and password of the device, so the user does not need to log in every
time they open the app. For any production environment, it is strongly recommended that the tablet is set in kiosk mode to protect the integrity
of the Cisco device.

It is also recommended to create a dedicated API user on the Cisco device, which can be deleted if the iPad is lost, without
impacting other integrations or admin users. This user can be of type **user**, it does not need to be an admin user type.

## Tips for testers


### URL based logins

You may frequently want to test to more than one device. Instead of logging out of the current device and in to the new every time,
you can connect by specifying host, username and password in the URL. This makes it easy to make bookmarks to multiple devices.

The link can be constructed like so:

https://linktoapp.com/?host=10.47.71.234&username=ipad&password=mysecret

### Insecure connection

It is possible, **but not recommended** to use insecure web socket (*ws*) instead of secure (*wss*) to the Cisco device.

* Set NetworkServices HTTP Mode: HTTP+HTTPS on the device
* Load the web app from an **insecure** http location
* When connecting from the table, specify insecure by using **ws://<ipaddress>** instead of **<ipadress>** for host

You should now be able to connect without any certificate issues.

## Creating your own certificates

TODO (How to create your own CA, and create and provision certificates for the iPad/controller and the Cisco device)


## Hosting the web app yourself

If you have a zip file of the app, simply unzip the content and host it on any web server.

If you want to build the app from the source code itself, make sure you have a recent version of Node and npm installed:

```bash
git clone <repo>
cd roomos-air
npm install
npm run build
```

The build should now be available in the `build` folder. You can zip this and move it to wherever you want to host the app.

## Contributing

You can fork the Git repository and create pull requests the normal way.

```bash
git clone <repo>
cd roomos-air
npm install
npm start
```

This will start a local web server with hot reloading. You can edit the React files, and the app should automatically reload when you
save the edits.


## Simulator / demo

The app also has a very basic device simulator built-in. This allows you to develop, test or demo the app without access to a physical Cisco device. Just connect with **simulator** as host and any username.
Unpair as usual to get back to the login page. You can also use the URL params to force this while developing, eg http://myapp.com/?host=simulator&username=ipad

## Useful links

- [Online hosted version](https://cisco-ce.github.com/airpad) - Github repo
- [GitHub repo](https://sqbu-github.cisco.com/roomos-air/roomos-air) - Potentially: open source on GitHub
- [React](https://react.dev/) - ui library
- [JSXAPI](https://github.com/cisco-ce/jsxapi) - SDK for talking xAPI with the Cisco device
- [xAPI](https://roomos.cisco.com/xapi) - public API reference
- Homemade [Momentum Design](https://momentum.design/) component library - based on
