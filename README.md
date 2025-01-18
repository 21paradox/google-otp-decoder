# google otp decoder

> decode google otp (Google Authenticator qrcode) and print to webpage & console

<xmp id="example">
  <button @click="scancamera">scan on camera</button>
  <span style="margin: 0 1rem 0 1rem">or upload image</span>
  <label>
    <input type="file" @input="uploadimg" />
  </label>
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
</xmp>

<script>   
  function decodeOtpUri(input) {
      let inputBase64 = input.replace('otpauth-migration://offline?data=', '')
      inputBase64 = unescape(inputBase64)

      const inputUtf8str = window.atob(inputBase64)

      const carr = inputUtf8str.split('')
      const buf = new Uint8Array(carr.length)
      carr.forEach((c, i) => {
        buf[i] = c.charCodeAt(0)
      })

      const pb = `syntax = "proto3";
option go_package = "github.com/kuingsmile/decodeGoogleOTP/migrationpayload";

message MigrationPayload {
  enum Algorithm {
    ALGO_INVALID = 0;
    ALGO_SHA1 = 1;
  }

  enum OtpType {
    OTP_INVALID = 0;
    OTP_HOTP = 1;
    OTP_TOTP = 2;
  }

  message OtpParameters {
    bytes secret = 1;
    string name = 2;
    string issuer = 3;
    Algorithm algorithm = 4;
    int32 digits = 5;
    OtpType type = 6;
    int64 counter = 7;
  }

  repeated OtpParameters otp_parameters = 1;
  int32 version = 2;
  int32 batch_size = 3;
  int32 batch_index = 4;
  int32 batch_id = 5;
}`
      const parsed = protobuf.parse(pb)
      const decodeRaw = parsed.root.MigrationPayload.decode(buf)
      const jspayload = parsed.root.MigrationPayload.toObject(decodeRaw, {
        enums: String,
        longs: String,
        bytes: String,
        defaults: true,
        arrays: true,
        objects: true,
        oneofs: true
      });
      return jspayload
    }


  new Vue({
    el: '#example',
    data() {
      return {
        otpUri: '',
        otpJson: '',
        qrScanner: null,
        camList: [
          {
            label: 'Environment Facing (default)',
            value: 'environment'
          },
          {
            label: 'user',
            value: 'user'
          }
        ]
      };
    },
    methods: {
      async changeCam(event) {
        if(this.qrScanner){
          this.qrScanner.setCamera(event.target.value)
        }
      },
      async scancamera() {
        const video = this.$refs.video;

        const qrScanner = new QrScanner(
          video,
          result => {
            console.log(result.data)
            this.otpUri = result.data
            const otpjson = decodeOtpUri(result.data)
            this.otpJson = JSON.stringify(otpjson)
          },
          {
            highlightScanRegion: true,
            highlightCodeOutline: true,
          },
        );
        this.qrScanner = qrScanner;

        await qrScanner.start();
        const cameras = await QrScanner.listCameras(true);

        cameras.forEach(camera => {
            this.camList.push({
              value: camera.id,
              label: camera.label
            });
        });
      },

      async uploadimg($event) {
        const file = $event.target.files[0]
        const result = await QrScanner.scanImage(file)
        this.otpUri = result
        const otpjson = decodeOtpUri(result)
        this.otpJson = JSON.stringify(otpjson)
      }
    }
  });
</script>

## develop

yarn dev
