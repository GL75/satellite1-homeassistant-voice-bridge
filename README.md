Work in progress

<h1 align="center">Satellite1 - Reroute Voice Input to Any External Media</h1>

<p align="center">
  <strong>A versatile architectural pattern to decouple a Home Assistant Voice Satellite's audio response from its local hardware.</strong>
</p>

<hr />

<h2>💡 The Challenge</h2>
<p>Many Home Assistant voice satellites (like those built on ESP32) attempt to play the TTS (Text-to-Speech) response through their own small speakers. However, enthusiasts often face hurdles when trying to use high-quality external speakers:</p>

<ul>
  <li><strong>Format Incompatibility:</strong> Some media players (like Sonos) cannot natively stream specific audio formats (FLAC/WAV) provided by certain TTS proxies.</li>
  <li><strong>Network Lag:</strong> Sending raw audio buffers over Wi-Fi can be brittle and prone to stuttering.</li>
  <li><strong>Limited Quality:</strong> Standard onboard ESP32 speakers often lack the fidelity desired for a premium smart home experience.</li>
</ul>

<h2>🚀 The Solution: The MQTT Text-Bridge</h2>
<p>Instead of sending <b>audio</b> from the satellite to the speaker, this project sends <b>text</b> from the satellite to Home Assistant via MQTT. Home Assistant then generates the TTS locally and sends it directly to the target media player.</p>



<h3>🔍 Technical Deep Dive: String vs. URL</h3>
<table width="100%">
  <thead>
    <tr>
      <th>Trigger Event</th>
      <th>Data Provided</th>
      <th>Why it matters for Rerouting</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>on_tts_start</code></td>
      <td><strong>text</strong> (String)</td>
      <td><strong>Used in this project.</strong> Provides raw reply text. It is format-agnostic and allows the target speaker's integration to handle audio rendering.</td>
    </tr>
    <tr>
      <td><code>on_tts_end</code></td>
      <td><strong>url</strong> (Link to .flac)</td>
      <td>Provides a link to the cached audio file. Passing this URL to external cloud-based speakers often results in "Media Format Not Supported" errors.</td>
    </tr>
  </tbody>
</table>

<hr />

<h2>🛠️ Implementation</h2>

<h3>1. ESPHome Configuration (Satellite Side)</h3>
<pre><code>
voice_assistant:
  id: va
  on_tts_start:
    - mqtt.publish:
        topic: "homeassistant/voice/reroute"
        payload: !lambda |-
          return "{\"message\": \"" + text + "\", \"target\": \"media_player.your_external_speaker\"}";
</code></pre>

<h3>2. Home Assistant Automation</h3>
<pre><code>
alias: "Satellite1 - Voice Response Reroute"
description: "Reroutes voice text to external media player via MQTT"
trigger:
  - platform: mqtt
    topic: "homeassistant/voice/reroute"
action:
  - variables:
      data: "{{ trigger.payload | from_json }}"
  - service: tts.cloud_say
    data:
      entity_id: "{{ data.target }}"
      message: "{{ data.message }}"
</code></pre>

<hr />

<h2>🌟 Key Benefits</h2>
<ul>
  <li><strong>Format Agnostic:</strong> Bypasses "Unsupported Format" errors on Sonos and other picky media players.</li>
  <li><strong>Ultra-Low Latency:</strong> Text travels faster than audio; the external speaker begins its stream immediately.</li>
  <li><strong>Room-Aware:</strong> Modify the payload to include different target entities based on which satellite was triggered.</li>
</ul>

<p align="right">
  <strong>Project Name:</strong> <code>ha-satellite1-voice-bridge</code><br>
  <strong>License:</strong> MIT
</p>
