# Mark-XLVII: Gemini API, Audio, and Remote Dashboard Setup

This runbook explains how to configure the Gemini API key, launch Mark-XLVII with local audio, and use its built-in web dashboard from another device.

## 1. Prerequisites

Mark-XLVII is installed at `/home/ubuntu/Mark-XLVII` with a Python virtual environment at `/home/ubuntu/Mark-XLVII/.venv`.

The application needs:

- Python 3.11 or 3.12.
- A working microphone and audio output on the host computer for local voice mode.
- A Gemini API key with access to the Gemini Live API. Create one at [Google AI Studio API Keys](https://aistudio.google.com/apikey).
- Network access between the remote browser and the computer running Mark-XLVII when using the dashboard directly.

Do not publish the API key, commit it to Git, or place it in a client-side web page.

## 2. Configure the Gemini API key

There are two supported methods.

### Method A: First-launch setup screen

Start the application:

```bash
cd /home/ubuntu/Mark-XLVII
source .venv/bin/activate
python main.py
```

On first launch, the initialization overlay asks for:

1. The Gemini API key.
2. The operating system. Linux is normally selected automatically.

Click **INITIALISE SYSTEMS**. The application saves the configuration locally as:

```text
/home/ubuntu/Mark-XLVII/config/api_keys.json
```

The file is deliberately ignored by Git because it contains secrets.

### Method B: Create the configuration file manually

Use this only on the host computer, never in a public repository:

```bash
cd /home/ubuntu/Mark-XLVII
mkdir -p config
cat > config/api_keys.json <<'JSON'
{
    "gemini_api_key": "PASTE_YOUR_GEMINI_KEY_HERE",
    "os_system": "linux"
}
JSON
chmod 600 config/api_keys.json
```

Replace the placeholder with the real key. Verify only the field names, not the secret value:

```bash
.venv/bin/python -c 'import json; print(sorted(json.load(open("config/api_keys.json"))))'
```

Expected output:

```text
['gemini_api_key', 'os_system']
```

## 3. Run Mark-XLVII with local audio

Activate the environment and start the application:

```bash
cd /home/ubuntu/Mark-XLVII
source .venv/bin/activate
python main.py
```

Mark-XLVII uses the computer microphone through `sounddevice` and plays generated audio through the host audio device. The default speech output engine is Edge TTS, which requires network access. The Gemini Live session itself also requires network access.

If Linux does not expose an input or output device, list the devices with:

```bash
.venv/bin/python -c 'import sounddevice as sd; print(sd.query_devices())'
```

For a quick audio-library check:

```bash
.venv/bin/python -c 'import sounddevice, soundfile, miniaudio; print("audio imports: ok")'
```

The default TTS configuration is `edgetts` with the `en-US-GuyNeural` voice. The TTS factory also supports `kokoro` and `elevenlabs` when their corresponding configuration and dependencies are supplied, but these are not needed for the default setup.

## 4. Use the built-in web dashboard on the local network

When Mark-XLVII is running, its dashboard starts automatically on port `8000` and binds to all interfaces. In the desktop UI:

1. Click **REMOTE CONTROL**.
2. A one-time six-character key and QR code appear.
3. On the phone, tablet, or another computer, scan the QR code or open the displayed URL.
4. Enter the one-time key if prompted.
5. The dashboard opens after pairing.

The one-time key expires after 10 minutes and is invalidated after use. The dashboard then uses a bearer session token and AES-256-CBC command encryption when the browser supports the bundled CryptoJS client.

To open it manually from another device on the same LAN, use the host IP printed by Mark-XLVII, for example:

```text
http://192.168.1.25:8000
```

Do not expose port 8000 directly to the public internet. The dashboard is intended for a trusted LAN or an encrypted tunnel.

## 5. Remote text commands and remote microphone audio

The dashboard supports:

- Sending text commands to Gemini Live.
- Uploading files for Mark-XLVII to process.
- Sending live microphone PCM audio from the browser to the host.
- Receiving JARVIS status and response events over WebSocket.

For remote microphone audio, the browser must grant microphone permission. Modern browsers require a secure context for `getUserMedia()`; `https://` is preferred.

### Preferred option: SSH tunnel

An SSH tunnel avoids exposing the dashboard to the LAN or internet. On the remote device, run:

```bash
ssh -N -L 8000:127.0.0.1:8000 USER@HOST
```

Then open this on the remote device:

```text
http://127.0.0.1:8000
```

Because the browser treats localhost as a secure context for microphone permissions, this is the simplest option for remote voice testing. Keep the SSH session open while using the dashboard.

### LAN option: HTTPS certificate

For direct LAN access with browser microphone permissions, place a certificate and private key at:

```text
/home/ubuntu/Mark-XLVII/config/certs/jarvis.crt
/home/ubuntu/Mark-XLVII/config/certs/jarvis.key
```

The application automatically serves HTTPS on port `8000` and an HTTPS manual-entry alias on port `8001`. With a self-signed certificate, accept the browser warning once on the remote device, then open the displayed HTTPS address.

A self-signed certificate is suitable only for a trusted private network. For a broader deployment, use a reverse proxy such as Caddy or Nginx with a trusted certificate, and restrict access with a VPN or identity-aware proxy.

### Last-resort browser workaround: HTTP

The dashboard contains an on-screen fallback that explains how to add its HTTP origin to Chrome's **Insecure origins treated as secure** flag. This is less secure than SSH or HTTPS and should only be used temporarily on a trusted network. Never use this workaround for a publicly reachable address.

## 6. Firewall and port checks

The dashboard attempts best-effort firewall setup for port 8000. If it cannot update the firewall, allow the port manually only on the trusted private network:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 8000 proto tcp
```

Adjust the subnet to match the LAN. Check that the server is listening:

```bash
ss -ltnp | grep ':8000'
```

If you use the HTTPS alias, also check port `8001`.

## 7. Run it as a background service

For a temporary background launch from an SSH session:

```bash
cd /home/ubuntu/Mark-XLVII
source .venv/bin/activate
nohup python main.py > mark-xlvii.log 2>&1 &
echo $! > mark-xlvii.pid
```

Stop it later with:

```bash
kill "$(cat /home/ubuntu/Mark-XLVII/mark-xlvii.pid)"
```

For a persistent Linux installation, create a user-level `systemd` service rather than running as root. A service should use an active graphical/audio session because the desktop UI, microphone, camera, and local audio output are OS-session resources.

## 8. Troubleshooting

| Symptom | Check |
|---|---|
| `API key invalid` | Confirm `config/api_keys.json` exists, contains `gemini_api_key`, and the key is active in Google AI Studio. |
| No microphone input | Run `sounddevice.query_devices()` and verify the host audio permissions and default input device. |
| No audio output | Verify the host output device and PulseAudio/ALSA session; test with local system audio first. |
| Dashboard not reachable | Confirm Mark-XLVII is running, port 8000 is listening, and the firewall allows the trusted LAN subnet. |
| Remote mic button does not work | Use an SSH tunnel or HTTPS, then grant microphone permission in the browser. |
| QR code expired | Click **REMOTE CONTROL** again to generate a fresh one-time key. |
| Dashboard says unavailable | Confirm `fastapi`, `uvicorn[standard]`, `cryptography`, and `python-multipart` are installed in `.venv`. |

## 9. Security checklist

- Keep `config/api_keys.json` at mode `600`.
- Do not commit the API key or share dashboard session tokens.
- Prefer an SSH tunnel or VPN over direct port exposure.
- Use HTTPS for browser microphone access when not using localhost through an SSH tunnel.
- Revoke paired devices from the dashboard if a phone or browser is lost.
- Do not forward port 8000 from a router to the public internet.
