# Changelog
## 5.0.1
🐛 After the first unanswered attempt the call job ended silently instead of redialing (bashio's strict mode stopped the script on an expected error)
🐛 Ringing / hang-up were not detected while pjsua was running because pjsua buffers its output, so calls were only ended by the fallback timer (ring_timeout + 10 s)
🐛 The registration check of the auto-answer client could not see the registration for the same reason


## 5.0.0
⚠️ Breaking: only `aarch64` and `amd64` are supported (like Home Assistant and the base image)
⚠️ Breaking: renamed to "NoPro200 - DSS VoIP Notifier" with slug `nopro200_dss_voip` and data folder `/share/nopro200_dss_voip`, so it can be installed next to the original add-on
⚠️ Breaking: `call_duration` now means a single call without redial. Leave it out to get the new redial behaviour
⬆️ Upgrades add-on base image to v21.0.7 (Alpine 3.24, s6-overlay v3), pjsua 2.17
⬆️ `config.json`/`build.json` replaced by `config.yaml`/`build.yaml`, option translations (en/de)
✨ Redial: rings for `ring_timeout` seconds (default 20 s ≈ 4 rings), hangs up and calls again until someone answers or `retry_timeout` (default 120 s) is reached
✨ `message_repeat`: play the message several times after answering (default 3)
✨ `redial_on_busy`: choose whether busy/declined calls are redialed (default on)
✨ All of these can also be set per call, plus `tts_engine` and `tts_language`
✨ Several calls can run at the same time
✨ `call_sip_uri` can be a plain number, it is completed with the domain of `caller_id_uri`
✨ Uses the current Home Assistant TTS API (TTS entities, e.g. `tts.piper`); the first TTS entity is used if `platform_tts` is not set
🐛 A call ends as soon as the other side hangs up, following calls are no longer queued
🐛 The SIP password was written to `/share/dss_voip/dss_pjsua.log`
🐛 A failing TTS request or audio conversion stopped the whole add-on
🐛 `pjsua_custom_options` longer than 200 characters were cut off by pjsua; en dashes (`–no-tcp`) are now fixed automatically
🐛 SIP parameters with spaces broke the pjsua config
🐛 The TTS file URL used the external Home Assistant URL that the add-on often cannot reach
🐛 `input` sent as JSON string instead of an object was not understood
🐛 The auto-answer SIP client is restarted if it stops, and its registration is actually checked
🐛 Wrong options in the README examples (sipgate, 3CX, NETelip)


## 4.0.0
⬆️ Upgrades add-on base image to v11.1.0
⬆️ Upgrades wget to 1.21.1-r1
⬆️ Upgrades coreutils to 8.32-r2


## 3.5.9
⬆️ Upgrades add-on base image to v10.0.1


## 3.5.7
⬆️ Upgrades add-on base image to v9.1.7


## 3.5.3
fix eol


## 3.5.1
⬆️ Upgrades add-on base image to v9.0.1
⬆️ Upgrades wget to 1.21.1-r1
⬆️ Upgrades coreutils to 8.32-r2


## 3.4.1
⬆️ Upgrades add-on base image to v8.0.3

