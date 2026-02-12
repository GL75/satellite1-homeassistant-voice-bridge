<h1 align="center">Satellite1 - Reroute Voice Outout to Any External Media in Home Assistant</h1>

<p align="center">
  <strong>A versatile architectural pattern to decouple the Satellite1's audio response from its local hardware. This allows you to reroute "Assist" responses to any media player integration (Sonos, HEOS, Volumio, etc.) using MQTT as a bridge between the Satellite1 and HA.</strong>
</p>

<hr />

<h2>💡 The Challenge</h2>
<p>The Satellite1 is design to play the TTS (Text-to-Speech) response through their own small speakers. However, enthusiasts may want to leverage other media players, in my case Sonos speakers installed in the ceiling. </p>

<h2>🚀 The Solution: The MQTT Text-Bridge</h2>
<p>Leveraging MQTT between the Satellite1 and Home assistant unlock few interesting possibilities. The ESP32 can publish MQTT topics at different stages of the intent process under the <b>voice_assistant:</b> section of the ESP32 yaml file. If you don't have an MQTT broker, it is quick an easy setup. Available as an add-on in HAOS. Has become my preferred method to communicated between platforms (HA, ESP32, Node-Red, Frigate, etc...</p>



<h3>🔍 Technical Deep Dive: String vs. URL</h3>
<table width="100%">
  <thead>
    <tr>
      <th>Trigger Event</th>
      <th>Data Provided (contained in variable 'x')</th>
      <th>Why it matters for Rerouting</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>on_tts_start</code></td>
      <td><strong>text</strong> (String)</td>
      <td><strong>Used in this project.</strong> Provides raw reply text. It is format-agnostic and allows the target to use a TTS mechanism of you choosing</td>
    </tr>
    <tr>
      <td><code>on_tts_end</code></td>
      <td><strong>url</strong> (Link to .flac)</td>
      <td>Provides a link to the cached audio file. Passing this URL to external cloud-based speakers often results in "Media Format Not Supported" errors.</td>
    </tr>
  </tbody>
</table>
<strong>Note:</strong> I started leverating <code>on_tts_end</code> to realize my targetted media player did not support flac files. Moving up the chain at <code>on_tts_start</code></td> allowed me to leverage a TTS of my choosing, compatible my targetted system
<hr />

<h2>🛠️ Implementation</h2>

<h3>1. ESPHome Configuration (Satellite Side)</h3>
Two sections to add/update : <code>mqtt:</code> and under <code>voice_assitant:</code>
<pre><code>
mqtt:
  broker: 192.168.0.xxx # Your MQTT Broker IP
  username: "mqtt"
  password: "strongpassword"
###
voice_assistant:
  on_tts_start:
    - mqtt.publish:
        topic: "home/voice/response_text"
        payload: !lambda |-
          return "{ \"reply\": \"" + x + "\", \"device\": \"media_player.living_room_ceiling_announce\" }";
</code></pre>

<h3>2. Home Assistant Automation</h3>
<pre><code>
alias: Voice Redirect
description: ""
triggers:
  - trigger: mqtt
    options:
      topic: home/voice/response_text
conditions: []
actions:
  - variables:
      data: "{{ trigger.payload | from_json }}"
    enabled: true
  - action: tts.cloud_say
    metadata: {}
    data:
      cache: false
      entity_id: "{{ data.device }}"
      message: "{{ data.reply }}"
mode: single

</code></pre>

<hr />

<h2>🌟 Key Benefits</h2>
<ul>
  <li><strong>WIP</strong> </li>
</ul>

<p align="right">
  <strong>Project Name:</strong> <code>ha-satellite1-voice-bridge</code><br>
  <strong>License:</strong> MIT
</p>
