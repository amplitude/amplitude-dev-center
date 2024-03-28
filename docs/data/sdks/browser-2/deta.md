---
title: Default Event Tracking Advanced
---

Amplitude's Default Event Tracking Advanced plugin extends the events and properties that Amplitude tracks by default, and enables Visual Tagger to enable you to define events based on elements on your site. For more information about Visual Tagger, see [Visual Tagger](https://help.amplitude.com/hc/en-us/articles/24094812669979-Visual-tag-editor).

--8<-- "includes/alpha-release.md"

## Installation

Default Event Tracking requires the latest versions of the Amplitude Browser SDK (@{$ browser.sdk.version $}) and the Default Event Tracking Advanced plugin (@{$ browser.plugin.deta.version $})

=== "Script Loader"
    ```html
    <script defer src="https://cdn.amplitude.com/libs/analytics-browser-@{$ browser.sdk.version $}-min.js.gz"></script>
    <script defer src="https://cdn.amplitude.com/libs/plugin-default-event-tracking-advanced-browser-@{$ browser.plugin.deta.version $}-min.js.gz"></script>
    ```
=== "NPM"
    ```bash
    npm install @amplitude/analytics-browser
    npm install @amplitude/plugin-default-event-tracking-advanced-browser@beta
    ```
=== "Yarn"
    ```bash
    yarn add @amplitude/analytics-browser
    yarn add @amplitude/plugin-default-event-tracking-advanced-browser@beta
    ```

## Initialize the plugin

The Amplitude Browser SDK supports a [plugin architecture](/data/sdk-plugins/) that enables features like Default Event Tracking Advanced. To enable the plugin, update your code with one of the following snippets, depending on your implementation. Both methods require that you define the Default Event Tracking Advanced plugin, then call `add()` to enable it.

=== "Script Loader"
    ```html
    <script type="module">
        window.amplitude.init(AMPLITUDE_API_KEY)
        const defaultEventTrackingAdvancedPlugin = window.amplitudeDefaultEventTrackingAdvancedPlugin.plugin();
        window.amplitude.add(defaultEventTrackingAdvancedPlugin);
    </script>
    ```
=== "NPM / Yarn"
    ```js
    import * as amplitude from '@amplitude/analytics-browser';
    import { defaultEventTrackingAdvancedPlugin } from '@amplitude/plugin-default-event-tracking-advanced-browser';

    amplitude.init(AMPLITUDE_API_KEY);
    amplitude.add(defaultEventTrackingAdvancedPlugin());
    ```

## Configuration

The Default Event Tracking Advanced plugin adds four settings that help you configure what the plugin tracks.

| <div class="big-column">Setting</div>                   | Default                                                                                              | Description                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `cssSelectorAllowlist`     | `['a','button','input','select','textarea','label','[data-amp-default-track]','.amp-default-track']` | String[]. Accepts one or more CSS selectors that define which elements on the page to track.         |
| `pageUrlAllowlist`         | undefined                                                                                            | `(string|RegExp)[]`. Defines the URL, URLs, or URL pattern on which Amplitude tracks default events |
| `shouldTrackEventResolver` | undefined                                                                                            | Function. Programatically determines if Amplitude should or shouldn't track an event.              |
| `dataAttributePrefix`      | `data-amp-track`                                                                                     | Allows the plugin to capture data attributes as an event property                                  |

```js
const plugin = defaultEventTrackingAdvancedPlugin({
  cssSelectorAllowlist: [
    '.amp-tracking',
    '[amp-tracking]'
  ],
  pageUrlAllowlist: [
    'https://amplitude.com',
    new RegExp('https://amplitude.com/blog/*')
  ],
});
```

By default, if you don't use these settings, Amplitude tracks the default selectors on all page on which you enable the plugin.

!!! info "Specifying CSS Selectors"
    When specify the CSS selectors to track, your selection overrides the default. To retain the default selectors import the `DEFAULT_CSS_SELECTOR_ALLOWLIST` and include it in your code.

    ```js
    import {
    DEFAULT_CSS_SELECTOR_ALLOWLIST,
    } from '@amplitude/plugin-default-event-tracking-advanced-browser';

    const selectors = [
        ...DEFAULT_CSS_SELECTOR_ALLOWLIST,
        '.class-of-a-thing-i-want-to-track',
    ];
    ```

## New events

When you enable the Default Event Tracking Advanced plugin, Amplitude sends two events, from which you can create Dynamic Events with Visual Tagger:

- `[Amplitude] Element Clicked`
- `[Amplitude] Element Changed`

These two events capture properties that describe the corresponding element and other context about the user's browser:

<!-- vale off-->
- `[Amplitude] Element ID`
- `[Amplitude] Element Class`
- `[Amplitude] Element Tag`
- `[Amplitude] Element Text` (Collected for `[Amplitude] Element Clicked`, only) 
- `[Amplitude] Element Href` (Collected for `[Amplitude] Element Clicked`, only)
- `[Amplitude] Element Position Left`
- `[Amplitude] Element Position Top`
- `[Amplitude] Viewport Height`
- `[Amplitude] Viewport Width`
- `[Amplitude] Page URL`
- `[Amplitude] Page Title`
- `[Amplitude] Element Selector`
- `[Amplitude] Element Attributes`
- `[Amplitude] Element Aria Label`
- `[Amplitude] Element Parent Label`
<!-- vale on-->

## Disable default event tracking

To disable default event tracking, remove the plugin from any pages that implement it, and set `defaultTracking: false` in the Amplitude initialization on that page.

=== "Script Loader"
    Remove the following lines of code:

    ```html
    <!-- load Amplitude DETA plugin -->
    <script defer src="https://cdn.amplitude.com/libs/plugin-default-event-tracking-advanced-browser-@{$ browser.plugin.deta.version $}-min.js.gz"></script>
    <!-- initialize Amplitude SDK and DETA plugin -->
    <script type="module">
    const defaultEventTrackingAdvancedPlugin = window.amplitudeDefaultEventTrackingAdvancedPlugin.plugin();
    window.amplitude.add(defaultEventTrackingAdvancedPlugin);
    </script>
    ```
=== "NPM / Yarn"
    Remove the plugin:

    ```bash
    // npm
    npm uninstall @amplitude/plugin-default-event-tracking-advanced-browser

    // yarn
    yarn remove @amplitude/plugin-default-event-tracking-advanced-browser
    ```

    Remove the initialization code:

    ```javascript
    // Remove the following lines of code
    import { defaultEventTrackingAdvancedPlugin } from '@amplitude/plugin-default-event-tracking-advanced-browser';

    amplitude.add(defaultEventTrackingAdvancedPlugin());
    ```

## Visual Tagger

Visual Tagger takes the properties the `[Amplitude] Element Clicked` and `[Amplitude] Element Changed` events capture and uses them to define Dynamic Events. For more information, see [Visual Tagger](https://help.amplitude.com/hc/en-us/articles/24094812669979-Visual-tag-editor)