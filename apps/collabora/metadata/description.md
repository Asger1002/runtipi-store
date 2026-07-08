# Collabora Online

Collabora Online is a LibreOffice-based online document editing server. It is the server component that Nextcloud Office connects to when editing ODT, ODS, DOCX, XLSX, PPTX, and related document formats in the browser.

## Nextcloud Office setup

1. Install this app and expose it on a real HTTPS hostname, for example:

   ```text
   https://collabora.catla-spica.ts.net/
   ```

2. In Nextcloud, install/enable **Nextcloud Office** and **Collabora Online - Built-in CODE Server** should remain disabled if you want to use this external server.

3. In Nextcloud go to:

   ```text
   Administration settings → Office
   ```

4. Choose **Use your own server** and enter:

   ```text
   https://collabora.catla-spica.ts.net
   ```

5. Save and test with an ODT/ODS file.

## Important: image choice

Collabora's public Docker image is `collabora/code`, the Collabora Online Development Edition (CODE). It is the freely available image suitable for home labs and testing. If you have access to a supported/stable Collabora Online image from Collabora, set the **Collabora image** field during install to that image instead.

This app is configured as an external WOPI document server; it is not the embedded Nextcloud CODE app.

## Allowed WOPI host

The **Allowed WOPI host** field controls which Nextcloud/Seafile host may connect to this Collabora instance. For your hs1 Nextcloud over Tailscale, use:

```text
https://nextcloud.catla-spica.ts.net:443
```

Add or change this value if you later want another WOPI client to use the same Collabora server.
