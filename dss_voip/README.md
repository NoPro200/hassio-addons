# Home Assistant Add-on: NoPro200 - DSS VoIP Notifier

![Version][version-shield] ![Supports aarch64 Architecture][aarch64-shield] ![Supports amd64 Architecture][amd64-shield] ![License][license-shield]

Make VoIP calls from Home Assistant and play a TTS message or an audio file
to whoever answers. Perfect for alarms, doorbells, water leak sensors or any
notification that must not be missed.

> [!NOTE]
> **Fork / continued development.** This add-on is a fork and further
> development of the original [DSS VoIP Notifier][original] by
> [SDeSalve][sdesalve].
>
> **Reworked with AI.** The rework (redial logic, parallel calls, new base
> image, new TTS API and the bug fixes listed in the [changelog](CHANGELOG.md))
> was done with the help of AI ([Claude Code][claude-code]). Please
> [open an issue][issue] if something does not work.

## Features

- 📞 **Redial until answered** – rings for a configurable time (default
  ~4 rings), hangs up if nobody answers and calls again, until someone
  picks up or the retry timeout is reached.
- 🔁 **Message repeat** – the message is played several times after the
  call has been answered.
- 🔀 **Parallel calls** – call several phones at the same time.
- ⏹️ **Ends immediately on hang-up** – no waiting for a fixed call duration,
  following calls are never queued.
- 🗣️ **Any TTS integration** – Google Translate, Piper, Cloud, ... (the
  current Home Assistant TTS API is used).
- 🎵 **Audio files** – play an mp3/wav/ogg/flac from a URL or from `/share`.
- ⚙️ **Per-call options** – ring time, retry timeout, repeats, redial on
  busy, TTS engine and language can be set for every call.
- 🩺 **Self-test** – optional registered SIP client that answers incoming
  calls with a test message.

## Quick start

1. Add the repository `https://github.com/NoPro200/hassio-addons` to the
   add-on store and install **NoPro200 - DSS VoIP Notifier**.
1. Configure your SIP account:

   ```yaml
   sip_parameters:
     caller_id_uri: sip:username@sipserver.com
     realm: "*"
     username: username
     password: password
   ```

1. Start the add-on and call it from an automation:

   ```yaml
   action: hassio.addon_stdin
   data:
     addon: xxxxxxxx_nopro200_dss_voip
     input:
       call_sip_uri: "+491701234567"
       message_tts: The alarm has been triggered!
   ```

   Replace `xxxxxxxx_nopro200_dss_voip` with the slug shown in the URL of the add-on
   page.

## How a call works

```text
dial ──► ringing ──(ring_timeout, ~4 rings)──► no answer ──► hang up
  ▲                                                             │
  └────────────── wait retry_delay ◄── retry_timeout left? ◄────┘
                                              │ no
                                              ▼
                                           give up

answered ──► message is played message_repeat times ──► hang up
           (ends immediately if the other side hangs up)
```

Busy or declined calls are redialed too, unless `redial_on_busy` is off.
Configuration or address errors (e.g. 401, 403, 404) stop immediately.

If `call_duration` is passed, a single call without redial is placed and
terminated after that many seconds – useful for pagers and intercoms.

## Options overview

| Option           | Default | Per call | Description |
| ---------------- | ------- | -------- | ----------- |
| `ring_timeout`   | `20`    | ✅ | Seconds to ring before hanging up and redialing |
| `retry_timeout`  | `120`   | ✅ | Stop redialing after this many seconds (`0` = dial once) |
| `retry_delay`    | `3`     | ✅ | Pause between two attempts |
| `message_repeat` | `3`     | ✅ | How often the message is played after answering |
| `redial_on_busy` | `true`  | ✅ | Redial when busy or declined |
| `max_call_time`  | –       | –  | Fixed talk time instead of `message_repeat` |
| `platform_tts`   | auto    | ✅ (`tts_engine`) | TTS entity or platform |
| `tts_language`   | –       | ✅ | TTS language, e.g. `de` |
| `call_duration`  | –       | only per call | Single call, terminated after N seconds (`-1` = audio length) |

Full documentation, all options and example configs for Fritz!Box, 3CX,
sipgate, Messagenet and more: **[DOCS.md](DOCS.md)**

## Differences to the original add-on

| | Original 4.0.0 | This fork 5.x |
| --- | --- | --- |
| No answer | Rings until `call_duration` is over | Hangs up after ~4 rings and redials |
| Several calls | Queued one after another | Run in parallel |
| Other side hangs up | Add-on waits for the remaining duration | Call ends immediately |
| Message after answer | Until timer ends | Repeated `message_repeat` times |
| TTS | Legacy API, `google_translate` only by default | Current API, any TTS entity |
| Base image | v11 (2022), s6-overlay v2 | v21, s6-overlay v3 |
| Architectures | aarch64, amd64, armhf, armv7, i386 | aarch64, amd64 |
| Slug / data folder | `dss_voip`, `/share/dss_voip` | `nopro200_dss_voip`, `/share/nopro200_dss_voip` |

Existing automations keep working: all old call parameters are still
supported. Only the meaning of `call_duration` changed slightly (it now
disables redial), and the add-on slug is different because it comes from
another repository.

## Installing next to the original

This add-on can be installed **at the same time** as the original
DSS VoIP Notifier: it uses its own slug (`nopro200_dss_voip`), its own data
folder (`/share/nopro200_dss_voip`) and random local SIP ports, so the two
add-ons do not interfere with each other. Handy for testing before switching.

## Logs & troubleshooting

- The add-on log shows every call with a `[Call N]` prefix.
- Detailed SIP log: `/share/nopro200_dss_voip/dss_pjsua.log`
- Audio conversion log: `/share/nopro200_dss_voip/dss_sox.log`
- Self-test client log: `/share/nopro200_dss_voip/dss_autoanswer.log`

Common problems:

- `408 (Request Timeout)` – try `pjsua_custom_options: --no-tcp`.
- Fritz!Box: `pjsua_custom_options: --ip-addr=<IP of Home Assistant>` is
  required.
- `No TTS engine found` – set up a TTS integration or set `platform_tts`.

## Credits & license

- Original add-on: [DSS VoIP Notifier][original] by [SDeSalve][sdesalve]
- Fork and continued development: [NoPro200][nopro200], reworked with AI
  ([Claude Code][claude-code])
- Uses [PJSIP/pjsua][pjsip] and [SoX][sox]

Licensed under [CC BY-NC 4.0](LICENSE.md), like the original – free for
non-commercial use with attribution.

[version-shield]: https://img.shields.io/badge/version-5.0.0-blue.svg
[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[license-shield]: https://img.shields.io/badge/license-CC%20BY--NC%204.0-lightgrey.svg
[original]: https://github.com/sdesalve/hassio-addons/tree/master/dss_voip
[sdesalve]: https://github.com/sdesalve
[nopro200]: https://github.com/NoPro200
[issue]: https://github.com/NoPro200/hassio-addons/issues
[claude-code]: https://claude.com/claude-code
[pjsip]: https://www.pjsip.org
[sox]: https://sox.sourceforge.net
