# Home Assistant Apple iCloud 2FA Fix

Unofficial patched version of the Home Assistant **Apple iCloud** integration.

This custom component fixes a problem where Home Assistant asks for an Apple verification code during setup or reauthentication, but Apple does not actually send or show a usable 2FA code.

The patch updates the iCloud authentication flow so that Home Assistant explicitly requests an Apple two-factor authentication code before showing the verification-code form.

## What This Fixes

In the original integration, users may see this kind of message after a few weeks:

```text
Authentication for your Apple Account has expired.
Your previously entered password is no longer working.
Update your password to keep using this integration.
```

After entering the password, Home Assistant asks for an Apple verification code. In many cases, however, no code appears on any Apple device, making reauthentication impossible without deleting and recreating the whole integration.

This custom component changes the 2FA flow to explicitly request the verification code from Apple.

## Status

This is an unofficial workaround until the fix is merged into Home Assistant Core.

It is based on the Home Assistant Core `icloud` integration with the following local changes:

- Uses `pyicloud==2.5.0`
- Calls `request_2fa_code()` before showing the verification-code form
- Avoids requesting a new Apple code on every failed code entry
- Keeps the existing Apple iCloud integration behavior otherwise unchanged

## Installation

### 1. Create a Backup

Before installing, create a full Home Assistant backup.

### 2. Copy the Custom Component

Copy the `icloud` folder into your Home Assistant `custom_components` directory:

```text
/config/custom_components/icloud/
```

After installation, the path should look like this:

```text
/config/custom_components/icloud/manifest.json
/config/custom_components/icloud/config_flow.py
/config/custom_components/icloud/account.py
```

If the `custom_components` directory does not exist yet, create it.

### 3. Restart Home Assistant

Restart Home Assistant completely.

A simple reload of integrations is not enough.

### 4. Reauthenticate Apple iCloud

Go to:

```text
Settings -> Devices & services -> Apple iCloud
```

Then reconfigure or reauthenticate the integration.

You should now receive an Apple verification code on your trusted Apple device.

## Important Notes

This custom component overrides the built-in Home Assistant `icloud` integration.

It does **not** replace or modify `icloud3`.

These are different integrations:

```text
/config/custom_components/icloud/   -> this patched Apple iCloud integration
/config/custom_components/icloud3/  -> iCloud3 custom integration
```

Do not copy this package into `icloud3`.

## Troubleshooting

### No Apple Code Appears

If no Apple verification code appears even after installing this custom component, remove the old iCloud session cache and restart Home Assistant:

```bash
mv /config/.storage/icloud /config/.storage/icloud.backup
```

Then restart Home Assistant and try to authenticate again.

### Invalid Password

Make sure you are using the password expected by the Home Assistant Apple iCloud integration.

Depending on the current Home Assistant and Apple behavior, this may be either your normal Apple Account password during MFA setup or an app-specific password.

### Home Assistant Still Uses the Built-In Integration

Check that the custom component path is correct:

```text
/config/custom_components/icloud/manifest.json
```

Also check the Home Assistant logs after restart. Home Assistant should load the custom integration from `custom_components`.

## Removing This Custom Component

To remove this workaround:

1. Delete the folder:

```text
/config/custom_components/icloud/
```

2. Restart Home Assistant.

Home Assistant will then use the built-in Apple iCloud integration again.

## Related Issues

This workaround is related to several long-running Home Assistant iCloud authentication issues, including:

- Apple iCloud integration asking for reauthentication every few weeks
- Apple verification code not being sent during reauthentication
- Repeated Apple 2FA prompts
- iCloud authentication problems after Apple-side login changes

## Disclaimer

This is not an official Home Assistant integration.

Use it at your own risk. Always create a backup before installing custom components.
