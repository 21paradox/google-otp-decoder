# google otp decoder

> decode google otp (Google Authenticator qrcode) and print to webpage & console

<div id="example">
  <button @click="scancamera">scan on camera</button>
  <span style="margin: 0 1rem 0 1rem">or upload image</span>
  <label>
    <input type="file" @input="uploadimg" />
  </label>
  <br />
  <br />

  <div>
    <b>Preferred camera:</b>
    <select id="cam-list" @input="changeCam">
        <option v-for="cam in camList" :value="cam.value">{{cam.label}}</option>
    </select>
  </div>

<video ref="video" autoplay playsinline style="max-width: 30rem"></video>

  <div style="word-wrap: break-word; white-space: pre-wrap; padding-bottom: 1rem;">{{otpUri}}</div>
  <div style="word-wrap: break-word; white-space: pre-wrap;">{{otpJson}}</div>
</div>

## develop

yarn dev
