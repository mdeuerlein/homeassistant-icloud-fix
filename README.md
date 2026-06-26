# Home Assistant Apple iCloud 2FA Fix

Unofficial patched version of the [Home Assistant **Apple iCloud**](https://www.home-assistant.io/integrations/icloud) integration.



This custom component fixes a problem where Home Assistant asks for an Apple verification code during setup or reauthentication, but Apple does not actually send or show a usable 2FA code.
It updates the iCloud authentication flow so that Home Assistant explicitly requests an Apple two-factor authentication code before showing the verification-code form.

There are several long-running **issues** related to this problem, including:
 
- home-assistant/core#67510 - [iCloud integration causes PIN code requests on Apple devices](https://github.com/home-assistant/core/issues/67510)
- home-assistant/core#101816 - [iCloud integration continuously pops MFA approval](https://github.com/home-assistant/core/issues/101816)
- home-assistant/core#128830 - [iCloud integration stopped working due to Apple SRP-6a implementation](https://github.com/home-assistant/core/issues/128830)
- home-assistant/core#160536 - [iCloud stopped working after updating to 2026.1](https://github.com/home-assistant/core/issues/160536)
- home-assistant/core#170959 - [Apple iCloud Integration not working for months](https://github.com/home-assistant/core/issues/170959)

**Relevant upstream PR:** home-assistant/core#171863

### What This Fixes

In the original integration, users may see this kind of message after a few weeks:

```text
Authentication for your Apple Account has expired.
Your previously entered password is no longer working.
Update your password to keep using this integration.
```

After entering the regular icloud account password, Home Assistant asks for an Apple verification code. In many cases, however, no code appears on any Apple device, making reauthentication impossible without deleting and recreating the whole integration.



This is an unofficial workaround based on the Home Assistant Core `icloud` integration with the following changes to explicitly request the verification code from Apple.:

- Uses `pyicloud==2.5.0`
- Calls `request_2fa_code()` before showing the verification-code form
- Avoids requesting a new Apple code on every failed code entry
- Keeps the existing Apple iCloud integration behavior otherwise unchanged

## Installation

#### 1. Create a Backup

Before installing, create a full Home Assistant backup.

#### 2. Install the Custom Component

You can install this either via **HACS** (recommended — you get automatic update notifications) or **manually** by copying the files.

##### Option A — HACS (Custom Repository)

This repository is intentionally **not** in the default HACS store because it overrides the built-in `icloud` integration. Add it as a custom repository instead:

1. In Home Assistant, open **HACS**.
2. Open the top-right **⋮ menu → Custom repositories**.
3. Enter the repository URL `https://github.com/mdeuerlein/homeassistant-icloud-fix`, choose type **Integration**, then click **Add**.
4. Search for **Apple iCloud 2FA Fix** in HACS and click **Download**.
5. Continue with step 3 (restart) below.

> **Note:** While installed, this replaces Home Assistant's built-in Apple iCloud integration (they share the same `icloud` domain).

##### Option B — Manual (ZIP)

Download [ha-icloud-fix.zip](https://github.com/mdeuerlein/homeassistant-icloud-fix/releases/latest/download/ha-icloud-fix.zip) from the [Releases](https://github.com/mdeuerlein/homeassistant-icloud-fix/releases/) page, unpack its content and copy the `icloud` folder into your Home Assistant `custom_components` directory:

```text
/config/custom_components/icloud/
```
After installation, the path should look like this:

```text
/config/custom_components/icloud/manifest.json
/config/custom_components/icloud/config_flow.py
/config/custom_components/icloud/account.py
...
```

If the `custom_components` or the `icloud` directory does not exist yet, create it. Do not copy/overwrite this package into `icloud3`.

#### 3. Restart Home Assistant

Restart Home Assistant completely! (*A simple reload of integrations is not enough.*)

#### 4. Reauthenticate Apple iCloud

Go to `Settings -> Devices & services -> Apple iCloud`

Then reconfigure or reauthenticate the integration and you should now receive an Apple verification code on one of your trusted Apple device.


## Troubleshooting

#### No Apple Code Appears

If no Apple verification code appears even after installing this custom component, remove the old iCloud session cache and restart Home Assistant:

```bash
mv /config/.storage/icloud /config/.storage/icloud.backup
```

Then restart Home Assistant and try to authenticate again.

#### Invalid Password

Make sure you are using the password expected by the Home Assistant Apple iCloud integration.

Depending on the current Home Assistant and Apple behavior, this may be either your normal Apple Account password during MFA setup or an app-specific password.

#### Home Assistant Still Uses the Built-In Integration

Check that the custom component path is correct:

```text
/config/custom_components/icloud/manifest.json
```

Also check the Home Assistant logs after restart. Home Assistant should load the custom integration from `custom_components`.

## How to remove/uninstall this fix

To remove this workaround:
1. Delete the folder `/config/custom_components/icloud/`
2. Restart your Home Assistant
3. The built-in Apple iCloud integration will be used again

## Disclaimer

This is not an official Home Assistant integration.

Use it at your own risk. Always create a backup before installing custom components.
