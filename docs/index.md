# Mark-XLVII

**Mark-XLVII** is a cross-platform, real-time voice AI assistant powered by Gemini Live.

This GitHub Pages site provides the public setup and operations documentation. The Python desktop runtime, Gemini Live connection, OS controls, and physical microphone/speaker must continue running on a computer or server; GitHub Pages hosts documentation only.

## Quick start

```bash
git clone https://github.com/Onielq/Mark-XLVII.git
cd Mark-XLVII
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

Configure the Gemini API key through the first-run setup screen. The local secret file is intentionally ignored by Git:

```text
config/api_keys.json
```

## Remote dashboard

When Mark-XLVII is running, open the dashboard from the host machine's trusted network address. Use the desktop UI's **REMOTE CONTROL** action for normal one-time pairing, or follow the [remote setup runbook](REMOTE_SETUP.md).

For browser microphone access, prefer HTTPS or an SSH tunnel. Do not publish port 8000 directly to the public internet.

## Documentation

- [Remote setup, Gemini API, local audio, and dashboard guide](REMOTE_SETUP.md)
- [Source repository on GitHub](https://github.com/Onielq/Mark-XLVII)

## Important limitation

GitHub Pages cannot run Mark-XLVII's Python process, Gemini Live WebSocket, desktop automation, or physical audio devices. It is suitable for a permanent documentation and landing site. Keep the assistant runtime on a trusted computer or persistent server.
