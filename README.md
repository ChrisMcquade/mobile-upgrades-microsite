# O2 Upgrades — Agentic POC Microsite

Concept microsite for the O2 Upgrades intent: presents the opportunity and the June 2026 utterance evidence, demos the GECX agent in **both POC modalities** (web = chat, app pattern = voice), and carries the **generative UI** pattern (A2UI-style client function tools) in a live playground and in the agent conversation.

Sibling to the QuickStart installation microsite; same lazy-session and `?qsdebug` patterns, migrated to the `ces-messenger` SDK (required for `registerClientSideFunction`).

## Deploy

1. Create a repo (e.g. `upgrades-microsite`), add `index.html` and this README, push.
2. Settings → Pages → deploy from branch → root. Done — the site is fully static.

The GenUI **playground works immediately with no agent connected** — it uses the same renderer the agent path uses, fed with sample payloads.

## Connect the agent

Build the demo agent first — the full walkthrough (agent settings, paste-ready instruction, all four tool definitions with schemas and mock Python code, channel setup, demo script) is in **`O2_Upgrades_Demo_Agent_Build_Booklet_v0_1.docx`**.

Then fill the `CONFIG` block at the top of the script in `index.html`:

```js
const CONFIG = {
  deploymentId: "projects/<project>/locations/eu/apps/<app-id>/deployments/<deployment-id>",
  chatTitle: "O2 Upgrades Agent",
  tools: {
    get_a2ui_catalog: "projects/<project>/locations/eu/apps/<app-id>/tools/<tool-id>",
    use_a2ui_component: "projects/<project>/locations/eu/apps/<app-id>/tools/<tool-id>"
  }
};
```

- `deploymentId` — from the agent's web-chat channel page in the CES console.
- Tool IDs — each client tool's **Integration** tab shows its full resource name inside a pre-filled `registerClientSideFunction` snippet.
- Leave a tool ID empty and that client function simply isn't registered; the agent falls back to text.

**Publish before testing here.** The simulator uses the latest draft; the deployed channel pins a **published version**. Save → simulator-verify → publish → point the deployment at the new version.

## Modalities

One deployment, one agent — the difference is widget configuration only (the `MODES` block):

| Attribute | Web (chat) | App pattern (voice) |
|---|---|---|
| `audio-input-mode` | `NONE` | `DEFAULT_OFF` |
| `audio-output-mode` | `DISABLED` | `DEFAULT_ON` |

`NONE` vs `DEFAULT_OFF` matters: `DEFAULT_OFF` only hides the mic button — the widget still opens the persistent bidirectional audio stream. On web that stream is unwanted overhead, so `NONE`. In voice mode the stream **is** the transport, and `DEFAULT_OFF` gives the tap-to-toggle mic (tap to talk, tap again to stop) with spoken replies.

Switching demo modes tears down and recreates the widget (audio attributes are read at element init), starting a fresh session. UK English synthesis voice: ***Pending*** — widget default until selected (`voice` attribute).

## Generative UI

Two client function tools carry the pattern:

- `get_a2ui_catalog` → returns the component catalogue (names, usage guidance, JSON schemas) declared in `A2UI_CATALOG`.
- `use_a2ui_component` → validates `component_name`, renders via the shared `renderComponent()`, inserts the HTML into the transcript (`insertMessage`), returns `RENDERED` immediately. Unknown/invalid → `ERROR` and the agent continues in text.

**Fire-and-forget by design**: the tool never holds its response open waiting for a tap — a card tap calls `sessionInput()` and arrives as the customer's next message ("I'd like to go with: Pixel 10"), so nothing can time out. Works identically in chat and voice sessions.

Components (POC set of two): `handset_comparison_card` (2–3 devices, every device carries an action) and `option_selector` (one question, 2–4 tappable options). Bespoke hand-rolled renderer — evaluating the official A2UI renderer is a separate spike per the A2UI overview doc.

## Diagnostics

Load with `?qsdebug=1` for the on-screen log (JS errors, promise rejections, failed fetches, `ces-messenger` events, client tool invocations). The same flag sets `enable-debugger` on the widget → `window.cesMessengerLogs`.

## Notes

- No session starts on page load — the widget is created on first launch click (token broker contacted then).
- No external assets; device images are CSS silhouettes.
- Utterance stats and quotes on the page come from the June 2026 AWS NLU export analysis (n=10,000); full data in `Upgrade_Utterances_Categorised_June2026.xlsx`.
