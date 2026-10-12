# Branded Persona mobile and desktop apps

Use this reference only when the user asks for an installable app dedicated to
one Persona. A Persona app is a package of the existing published Persona
experience, not a second Persona and not a fork of its chat configuration.

## Canonical manifest

The Persona root repository owns exactly one cross-platform manifest:

`assets/persona-app-config.json`

Use schema version 2. Never introduce `desktop-app-config.json`. For an
unmigrated repository, tooling may read legacy `assets/mobile-app-config.json`,
but a repository containing both files is invalid. Migrate by writing the v2
file and deleting the legacy file in the same commit.

```json
{
  "schemaVersion": 2,
  "personaRef": "published-page-id",
  "pageSlug": "lane",
  "appKey": "lane",
  "displayName": "Lane",
  "experience": {
    "preset": "persona-chat",
    "requireSignIn": true,
    "impactLabel": "ROI Impact"
  },
  "branding": {
    "accentColor": "#2563EB",
    "theme": {
      "variant": "lane",
      "defaultThemeMode": "light",
      "fontFamily": null,
      "colors": {
        "light": {
          "canvas": "#F8FAFC",
          "surface": "#FFFFFF",
          "sidebar": "#F1F5F9",
          "text": "#0F172A",
          "accent": "#2563EB",
          "userBubble": "#2563EB"
        },
        "dark": {
          "canvas": "#0F172A",
          "surface": "#111827",
          "sidebar": "#020617",
          "text": "#F8FAFC",
          "accent": "#2563EB",
          "userBubble": "#2563EB"
        }
      }
    }
  },
  "mobile": {
    "platforms": ["android", "ios"],
    "android": { "applicationId": "com.example.lane" },
    "ios": { "bundleId": "com.example.lane" }
  },
  "desktop": {
    "platforms": ["mac", "win"],
    "appId": "com.example.desktop.lane",
    "protocolScheme": "gabriel-persona-lane",
    "shortcut": "CommandOrControl+Alt+L",
    "updateChannel": "lane"
  }
}
```

The actual `personaRef` is the deployed page ID returned by page creation. It
is not a portable resource key, repository UUID, action ID, or guessed slug.
`pageSlug` must match the published Persona. The desktop package resolves both
and fails closed when the installed identity does not match the published
Persona.

The manifest contains presentation and packaging metadata only. Never put API
keys, store credentials, certificates, signing material, conversations, model
weights, local model paths, device preferences, or build-repository bindings in
it. Keep versioned source artwork under `assets/mobile/`; both packagers may use
it until a later cross-platform asset migration.

## Authoring sequence

1. Create the page and keep its page ID and confirmed `pageSlug`.
2. Finish `assets/chat-config.json` and `assets/chat-app.json`. App navigation,
   flagship action, playbooks, routines, impact, and theme come from the same
   published Chat App; do not duplicate a mobile or desktop menu in this file.
3. Ask which platforms are required and confirm the long-lived identifiers.
   Identifiers, protocol schemes, update channels, and shortcuts must be unique
   across every app the user intends to install together.
4. Derive branding from the authored published Persona. If explicit theme
   seeds are included, provide all six light and all six dark colors and make
   `branding.accentColor` equal `branding.theme.colors.light.accent`.
5. Author and validate the canonical manifest in the Persona root repository.
   Request the `mobile-app-builder` child skill for the complete validation and
   mobile build contract.
6. Commit the manifest and artwork before publishing the workspace candidate.
   Persona apps consume the same passing published candidate as web chat.
7. In the signed-in product, open the Persona’s **Publish → Persona Apps** tab.
   Review the no-code form, the changed JSON paths, and the generated JSON.
   **Save and register configuration** writes the page-bound configuration and
   synchronizes `assets/persona-app-config.json` to Persona Git.

Saving registers the manifest with Gabriel for that Persona. It does not reserve
an ID in an external store and does not compile an application.

## Mobile build and vendor registration

In **Publish → Persona Apps**:

1. Save the manifest before selecting a build.
2. Connect the owner’s GitHub account.
3. Select or create a writable private repository owned by that user.
4. Select a released mobile package, save the binding, then explicitly choose
   **Build mobile test app**.
5. Test sign-in, fixed-Persona enforcement, Coach, the flagship action,
   sessions, impact, notifications, models, voice, attachments, and logout.

Current CI creates Android test APKs and iOS simulator archives. It does not
publish a store listing or produce a signed iPhone release. Before public
distribution, the owner must create matching Google Play and App Store Connect
records, configure signing/certificates and store metadata, and run the release
pipeline. Do not ask the owner to put vendor credentials or signing files in the
Persona manifest or paste them into chat.

No model weights are bundled or downloaded by app creation, launch, sign-in,
Persona activation, catalog browsing, or model selection. A downloadable local
model is acquired only after an explicit Install confirmation; Apple Foundation
Models remain OS-managed.

## Desktop build and vendor registration

From the marketplace checkout, use the canonical file directly:

```bash
./desktop/persona-app.sh --config /path/to/persona/assets/persona-app-config.json --run
./desktop/persona-app.sh --config /path/to/persona/assets/persona-app-config.json --pack --platform mac
./desktop/persona-app.sh --config /path/to/persona/assets/persona-app-config.json --dist --platform win
```

The driver creates a clean snapshot under the configured builds root and stamps
the product name, executable name, app ID, protocol, shortcut, update channel,
icons, tray identity, initial route, and isolated application data from the
manifest. The branded app is pinned to the configured Persona; it never exposes
the generic Persona picker. `/desktop/cursor` remains the generic multi-Persona
app and `/desktop/persona` is the branded route.

For distribution, the owner still configures platform signing, macOS
notarization, Windows signing/installer identity, and an update feed appropriate
to the declared channel. A manifest `updateChannel` separates app releases; it
does not create or host an update service.

## Acceptance checklist

- The repository contains canonical v2 only, with no legacy or desktop-specific
  duplicate manifest.
- `personaRef`, `pageSlug`, authored presentation, and packaged identity match
  the targeted published Persona.
- Mobile IDs, desktop ID, protocol, update channel, and shortcut are unique.
- The no-code Publish UI and exported JSON match the committed manifest.
- Both mobile targets and both desktop targets validate; unsupported targets
  are omitted explicitly.
- The branded app cannot switch Persona or accept a cross-Persona deep link.
- Successful local-model turns make no cloud-model request; fallback requires
  one-turn consent.
- Release packages and fresh app data contain no downloadable model weights.
- Test builds, vendor registration, signing, and store publication are reported
  as separate states; never claim public registration from a saved manifest.

## Shared web and native workspace

A branded mobile/desktop shell must consume the same enabled canonical Chat App as the signed-in web persona. Use the [shared dashboard format](../../chat-app-builder/references/dashboard-first.md): Home with Meet Persona review coach and domain tabs, a minimal sidebar, separate guided history/sidebar Playbooks, Coach milestones, Signals/Schedule/routines and Data Feed. Never duplicate these managers as Home tabs. Registering `persona-app-config.json` alone cannot supply this workspace. Validate both canonical app and persona-app manifest and check the Home/coach transition on each requested platform.
