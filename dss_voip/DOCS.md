# NoPro200 - DSS VoIP Notifier

Make VoIP calls from Home Assistant: a text is turned into speech by any
Home Assistant TTS integration (or an audio file is used), a SIP number is
called and the message is played to whoever answers.

Features:

- Calls ring for a configurable time (default ~4 rings). If nobody answers,
  the add-on hangs up and calls again until someone answers or the retry
  timeout is reached.
- The message is repeated a configurable number of times after answering.
- Several calls can run at the same time.
- A call ends immediately when the other side hangs up.

## Installation

1. Add the repository `https://github.com/NoPro200/hassio-addons` to your
   add-on store.
1. Install the "NoPro200 - DSS VoIP Notifier" add-on.
1. Configure at least `caller_id_uri`, `username` and `password`.
1. Set up a TTS integration in Home Assistant (e.g. Google Translate, Piper).
1. Start the add-on and check its log.

## Configuration

**Note**: _Restart the add-on after changing the configuration._

```yaml
sip_parameters:
  caller_id_uri: sip:username@sipserver.com
  realm: "*"
  username: username
  password: password
ring_timeout: 20
retry_timeout: 120
message_repeat: 3
```

### Option `sip_parameters`

| Key              | Description |
| ---------------- | ----------- |
| `caller_id_uri`  | (Required) SIP URL of the caller account (From header), e.g. `sip:username@sipserver.com`. |
| `username`       | Authentication user ID, if required by your SIP server. |
| `password`       | Authentication password, if required by your SIP server. |
| `realm`          | Authentication realm. Default `*` (answer to any realm). |
| `sip_server_uri` | (Optional) Registrar URL, e.g. `sip:sipserver.com`. If set, the add-on registers a SIP client that answers incoming calls with a test message, so you can check the system status. Log: `/share/nopro200_dss_voip/dss_autoanswer.log`. |

### Option `ring_timeout`

Seconds the phone may ring before the add-on hangs up and redials.
Default `20` (about 4 rings). The timer starts when the other side signals
ringing.

### Option `retry_timeout`

Seconds after which redialing stops if nobody answers. Default `120`.
`0` = dial only once. An answered call is never cut by this timeout.

### Option `retry_delay`

Pause in seconds between two attempts. Default `3`.

### Option `message_repeat`

How often the message is played after the call has been answered.
Default `3`.

### Option `redial_on_busy`

Also redial when the line is busy or the call is declined (SIP 486/600/603).
Default `true`. Configuration or address errors (e.g. 401, 403, 404) never
trigger a redial.

### Option `max_call_time`

Fixed time in seconds an answered call stays up, instead of
`message_repeat`. Not set or `-1` = depends on `message_repeat`.

### Option `platform_tts`

TTS engine to use: a TTS entity like `tts.google_translate_en_com` /
`tts.piper` or a legacy platform name like `google_translate`. If not set,
the first TTS entity found in Home Assistant is used.

### Option `tts_language`

Optional language for TTS, e.g. `de` or `en`. If not set, the default of
the TTS engine is used.

### Option `pjsua_custom_options`

Additional command line options for pjsua, see the
[pjsua documentation][pjsuaman]. Do not set a fixed `--local-port`,
otherwise parallel calls will fail.

### Options `sox_custom_options_input_file` / `sox_custom_options_output_file`

Additional [SoX][soxman] options for the input / output file. The output is
converted to 16 kHz mono 16 bit by default.

## How to use

Call the add-on from an automation or script:

```yaml
action: hassio.addon_stdin
data:
  addon: xxxxxxxx_nopro200_dss_voip
  input:
    call_sip_uri: sip:+491701234567@sipserver.com
    message_tts: The alarm has been triggered!
```

Replace `xxxxxxxx_nopro200_dss_voip` with the slug shown in the URL of the add-on
page in Home Assistant (it depends on the repository).

### Call parameters

| Parameter        | Description |
| ---------------- | ----------- |
| `call_sip_uri`   | (Required) SIP URL to call, e.g. `sip:+491701234567@sipserver.com`. A plain number (`+491701234567`) is completed with the domain of `caller_id_uri`. |
| `message_tts`    | Text to speak. Required if `audio_file_url` is not set. |
| `audio_file_url` | URL (`http://`, `https://`) or local path (e.g. `/share/doorbell.mp3`) of an audio file (mp3, wav, ogg, flac). Required if `message_tts` is not set. Files from Home Assistant's `www` folder can be used with `http://homeassistant:8123/local/file.mp3`. |
| `tts_engine`     | TTS engine for this call (overrides `platform_tts`). |
| `tts_language`   | TTS language for this call (overrides `tts_language`). |
| `message_repeat` | How often the message is played after answering (overrides `message_repeat`). |
| `redial_on_busy` | `true` / `false`, redial if busy or declined (overrides `redial_on_busy`). |
| `ring_timeout`   | Ring time per attempt in seconds (overrides `ring_timeout`). |
| `retry_timeout`  | Redial for up to this many seconds (overrides `retry_timeout`). |
| `retry_delay`    | Pause between attempts in seconds (overrides `retry_delay`). |
| `call_duration`  | Optional. If set, a **single** call is placed (no redial) and terminated after this many seconds, counted from dialing. `-1` = length of the audio file. Meant for auto-answering systems like pagers and intercoms. |

Example with all options:

```yaml
action: hassio.addon_stdin
data:
  addon: xxxxxxxx_nopro200_dss_voip
  input:
    call_sip_uri: "**610"
    message_tts: Somebody is at the door.
    tts_language: en
    message_repeat: 2
    redial_on_busy: false
    ring_timeout: 25
    retry_timeout: 300
```

To call several phones at once, send one service call per phone.

**Note**: The audio file should not be longer than 2 minutes 20 seconds,
pjsua may break up the voice after that time.

## Example config for some VoIP providers

### Fritz!Box

```yaml
sip_parameters:
  caller_id_uri: sip:username@fritz.box:5060
  realm: "*"
  username: username
  password: password
pjsua_custom_options: --ip-addr=HOME_ASSISTANT_IP_ADDRESS
```

`call_sip_uri` must end with `@fritz.box:5060`, e.g.
`sip:+491701234567@fritz.box:5060` (or just pass the number). Internal
phones can be called with `**610` etc. `--ip-addr` is mandatory, replace
`HOME_ASSISTANT_IP_ADDRESS` with the LAN IP of Home Assistant.

### [Pbxes.com][pbxesurl]

```yaml
sip_parameters:
  caller_id_uri: sip:extension@pbxes.com
  realm: "*"
  username: extension
  password: password
```

### [3CX PBX][3cxurl]

```yaml
sip_parameters:
  caller_id_uri: sip:extension@domain.3cx.com.au
  realm: "*"
  username: AuthenticationID
  password: AuthenticationPassword
pjsua_custom_options: --no-tcp
```

Use the Authentication ID and password from the extension settings for
username and password, but the SIP extension number for the caller ID URI.

### [Messagenet][messageneturl]

```yaml
sip_parameters:
  caller_id_uri: sip:numerointerno@sip.messagenet.it
  realm: "*"
  username: numerointerno
  password: password
pjsua_custom_options: --outbound=sip:sip.messagenet.it:5061;lr
```

### [FreeVoipDeal][freevoipdeal] / other Dellmont/Betamax providers

```yaml
sip_parameters:
  caller_id_uri: sip:username_or_phonenumber@sip.freevoipdeal.com
  realm: "*"
  username: username
  password: password
pjsua_custom_options: --no-tcp
```

`--no-tcp` is mandatory, otherwise you get `408 (Request Timeout)`.

### [2talk][2talkurl]

```yaml
sip_parameters:
  caller_id_uri: sip:areacodeandnumber@2talk.co.nz:5060
  realm: "*"
  username: areacodeandnumber
  password: secretpassword
```

### sipgate

```yaml
sip_parameters:
  caller_id_uri: sip:12345678@sipgate.de
  realm: "*"
  username: "12345678"
  password: MYPASS
  sip_server_uri: sip:sipgate.de
pjsua_custom_options: --no-tcp
```

### NETelip

```yaml
sip_parameters:
  caller_id_uri: sip:USERVOIP@sip.netelip.com:9060
  realm: "*"
  username: USERVOIP
  password: PASSWORD
  sip_server_uri: sip:sip.netelip.com:9060
pjsua_custom_options: --no-tcp --outbound=sip:sip.netelip.com:9060;lr
```

## Logs

- Add-on log: overview of every call (`[Call N]` prefix).
- `/share/nopro200_dss_voip/dss_pjsua.log`: detailed SIP log of all calls.
- `/share/nopro200_dss_voip/dss_sox.log`: audio conversion log.
- `/share/nopro200_dss_voip/dss_autoanswer.log`: log of the registered SIP client.

## License

Fork and further development of the original [DSS VoIP Notifier][original]
by [SDeSalve][sdesalve], reworked with the help of AI (Claude Code).
Licensed under [CC BY-NC 4.0](LICENSE.md).

[pjsuaman]: https://docs.pjsip.org/en/latest/specific-guides/other/cli_cmd.html
[soxman]: https://sox.sourceforge.net/sox.html
[pbxesurl]: https://www.pbxes.com
[3cxurl]: https://www.3cx.com/
[messageneturl]: https://messagenet.com/it
[freevoipdeal]: https://www.freevoipdeal.com
[2talkurl]: https://www.2talk.co.nz/
[original]: https://github.com/sdesalve/hassio-addons/tree/master/dss_voip
[sdesalve]: https://github.com/sdesalve
