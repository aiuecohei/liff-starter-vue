<template>
  <div class="home">
    <div v-if="liffAppContent">
      <!-- ACTION BUTTONS -->
      <div class="buttonGroup">
        <div class="buttonRow">
          <button @click="openWindow()">Open External Window</button>
          <button @click="closeWindow()">Close LIFF App</button>
        </div>
        <div class="buttonRow">
          <button @click="scanQrCode()">Open QR Code Reader</button>
          <button @click="sendMessage()">Send Message</button>
        </div>
        <div class="buttonRow">
          <button @click="getAccessToken()">Get Access Token</button>
          <button @click="getProfile()">Get Profile</button>
        </div>
        <div class="buttonRow">
          <button v-if="isInClient" @click="shareTargetPicker()">Open Share Target Picker</button>
        </div>
      </div>
      <div>{{ shareTargetPickerMessage }}</div>
      <!-- ACCESS TOKEN DATA -->
      <div v-if="showAccessToken" class="textLeft">
        <h2>Access Token</h2>
        <a href="#" @click="toggleAccessToken()">Close Access Token</a>
        <table>
          <tr>
            <th>accessToken</th>
            <td>{{ accessToken }}</td>
          </tr>
          <tr>
            <th>IDToken</th>
            <td>{{ idToken }}</td>
          </tr>
          <tr>
            <th>DecodedIDToken</th>
            <td>{{ decodedIdToken }}</td>
          </tr>
        </table>
      </div>
      <!-- SCAN QR RESULT -->
      <div v-if="showQrCodeResult" class="textLeft">
        <h2>QR Code reader</h2>
        <a href="#" @click="toggleQrCodeReader()">Close QR Code Reader Result</a>
        <table>
          <tr>
            <th>scanCode Result</th>
            <td>{{ qrCodeResult }}</td>
          </tr>
        </table>
      </div>
      <!-- PROFILE INFO -->
      <div v-if="showProfile" class="textLeft">
        <h2>Profile</h2>
        <a href="#" @click="toggleProfileData()">Close Profile</a>
        <div><img alt="Profile Picture" :src="profile.pictureUrl" /></div>
        <table>
          <tr>
            <th>userId</th>
            <td>{{ profile.userId }}</td>
          </tr>
          <tr>
            <th>displayName</th>
            <td>{{ profile.displayName }}</td>
          </tr>
          <tr>
            <th>statusMessage</th>
            <td>{{ profile.statusMessage }}</td>
          </tr>
        </table>
      </div>
      <!-- LIFF DATA -->
      <div>
        <h2 class="textLeft">LIFF Data</h2>
        <table>
          <tr>
            <th>OS</th>
            <td class="textLeft">{{ deviceOS }}</td>
          </tr>
          <tr>
            <th>Language</th>
            <td class="textLeft">{{ language }}</td>
          </tr>
          <tr>
            <th>LIFF SDK Version</th>
            <td class="textLeft">{{ sdkVersion }}</td>
          </tr>
          <tr>
            <th>LINE Version</th>
            <td class="textLeft">{{ lineVersion }}</td>
          </tr>
          <tr>
            <th>isInClient</th>
            <td class="textLeft">{{ isInClient }}</td>
          </tr>
          <tr>
            <th>isLoggedIn</th>
            <td class="textLeft">{{ isLoggedIn }}</td>
          </tr>
        </table>
      </div>
      <!-- LOGIN LOGOUT BUTTONS -->
      <div class="buttonGroup">
        <button v-if="!isInClient" :disabled="isLoggedIn" @click="liffLogin()">Log in</button>
        <button v-if="!isInClient" :disabled="!isLoggedIn" @click="liffLogout()">Log out</button>
      </div>
      <div>
        <div>{{ isInClientMessage }}</div>
        <div>
          <p>Available LIFF methods vary depending on the browser you use to open the LIFF app.</p>
          <p>
            Please refer to the <a href="https://developers.line.biz/en/reference/liff/#initialize-liff-app">API reference page</a> for more
            information.
          </p>
        </div>
      </div>
    </div>
    <!-- LIFF ID ERROR -->
    <!--
    <div id="liffIdErrorMessage" class="hidden">
      <p>You have not assigned any value for LIFF ID.</p>
      <p>
        If you are running the app using Node.js, please set the LIFF ID as an environment variable in your Heroku account follwing the
        below steps:
      </p>
      <code id="code-block">
        <ol>
          <li>Go to `Dashboard` in your Heroku account.</li>
          <li>Click on the app you just created.</li>
          <li>Click on `Settings` and toggle `Reveal Config Vars`.</li>
          <li>Set `MY_LIFF_ID` as the key and the LIFF ID as the value.</li>
          <li>Your app should be up and running. Enter the URL of your app in a web browser.</li>
        </ol>
      </code>
      <p>If you are using any other platform, please add your LIFF ID in the <code>index.html</code> file.</p>
      <p>
        For more information about how to add your LIFF ID, see
        <a href="https://developers.line.biz/en/reference/liff/#initialize-liff-app">Initializing the LIFF app</a>.
      </p>
    </div>
    -->
    <!-- LIFF INIT ERROR -->
    <div v-if="liffInitErrorMessage">
      <p>Something went wrong with LIFF initialization.</p>
      <p>
        LIFF initialization can fail if a user clicks "Cancel" on the "Grant permission" screen, or if an error occurs in the process of
        <code>liff.init()</code>.
      </p>
    </div>
    <!-- NODE.JS LIFF ID ERROR -->
    <!--
    <div id="nodeLiffIdErrorMessage" class="hidden">
      <p>Unable to receive the LIFF ID as an environment variable.</p>
    </div>
    -->
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import liff from '@line/liff';

@Component({
  components: {},
})
export default class Home extends Vue {
  public language: string | null = null;
  public sdkVersion: string | null = null;
  public lineVersion: string | null = null;
  public isInClient: boolean = false;
  public isLoggedIn: boolean = false;
  public deviceOS: string | null = null;
  public showAccessToken: boolean = false;
  public accessToken: string | null = null;
  public idToken: string | null = null;
  public showQrCodeResult: boolean = false;
  public qrCodeResult: string | null = null;
  public decodedIdToken: object | null = null;
  public context: object | null = null;
  public showProfile: boolean = false;
  public profile: object | null = null;
  public friendShip: object | null = null;
  public shareTargetPickerMessage: string | null = null;
  public isInClientMessage: string | null = null;
  public liffAppContent: boolean = true;
  public liffInitErrorMessage: boolean = false;

  public created() {
    this.initializeLiff();
  }

  public initializeLiff() {
    liff
      .init({
        liffId: process.env.LIFF_ID,
      })
      .then(() => {
        // start to use LIFF's api
        this.initializeApp();
      })
      .catch((err) => {
        console.error(err);
        this.liffAppContent = false;
        this.liffInitErrorMessage = true;
      });
  }
  public initializeApp() {
    this.displayLiffData();
    this.displayIsInClientInfo();
  }
  async displayLiffData() {
    this.language = await liff.getLanguage();
    this.sdkVersion = await liff.getVersion();
    this.lineVersion = await liff.getLineVersion();
    this.isInClient = await liff.isInClient();
    this.isLoggedIn = await liff.isLoggedIn();
    this.deviceOS = await liff.getOS()!;
    this.context = await liff.getContext();
  }
  displayIsInClientInfo() {
    if (liff.isInClient()) {
      this.isInClientMessage = 'You are opening the app in the in-app browser of LINE.';
    } else {
      this.isInClientMessage = 'You are opening the app in an external browser.';
    }
  }
  public openWindow() {
    liff.openWindow({
      url: 'https://line.me',
      external: true,
    });
  }
  closeWindow() {
    if (!liff.isInClient()) {
      this.sendAlertIfNotInClient();
    } else {
      liff.closeWindow();
    }
  }
  sendMessage() {
    if (!liff.isInClient()) {
      this.sendAlertIfNotInClient();
    } else {
      liff
        .sendMessages([
          {
            type: 'text',
            text: "You've successfully sent a message! Hooray!",
          },
        ])
        .then(() => {
          alert('Message sent');
        })
        .catch((err) => {
          alert('Error sending message: ' + err);
          console.error(err);
        });
    }
  }
  scanQrCode() {
    if (!liff.isInClient()) {
      this.sendAlertIfNotInClient();
    } else {
      if (liff.scanCode) {
        liff
          .scanCode()
          .then((result) => {
            this.qrCodeResult = JSON.stringify(result);
            this.toggleQrCodeReader();
          })
          .catch((err) => {
            alert('Error scan code: ' + err);
            console.error(err);
          });
      }
    }
  }
  getAccessToken() {
    if (!liff.isLoggedIn() && !liff.isInClient()) {
      alert('To get an access token, you need to be logged in. Please tap the "login" button below and try again.');
    } else {
      this.accessToken = liff.getAccessToken();
      // LIFFアプリをチャネルに追加するときに、openidスコープを選択しなかった場合は、nullが返されます。
      this.idToken = liff.getIDToken();
      // LIFFアプリをチャネルに追加するときに、openidスコープを選択しなかった場合は、nullが返されます。
      this.decodedIdToken = liff.getDecodedIDToken();
      this.toggleAccessToken();
    }
  }
  getProfile() {
    liff
      .getProfile()
      .then((profile) => {
        this.profile = profile;
        this.toggleProfileData();
      })
      .catch((err) => {
        alert('Error getting profile: ' + err);
        console.error(err);
      });
  }
  shareTargetPicker() {
    if (!liff.isInClient()) {
      this.sendAlertIfNotInClient();
    } else if (liff.isApiAvailable('shareTargetPicker')) {
      liff
        .shareTargetPicker([
          {
            type: 'text',
            text: 'Hello, World!',
          },
        ])
        .then(() => {
          this.shareTargetPickerMessage = 'Share target picker was launched.';
        })
        .catch((res) => {
          this.shareTargetPickerMessage = 'Failed to launch share target picker.';
          console.error(res);
        });
    } else {
      alert('liff.isApiAvailable("shareTargetPicker") is false');
    }
  }
  liffLogin() {
    if (!liff.isLoggedIn()) {
      // set `redirectUri` to redirect the user to a URL other than the front page of your LIFF app.
      liff.login();
    }
  }
  liffLogout() {
    if (liff.isLoggedIn()) {
      liff.logout();
      location.reload();
    }
  }
  sendAlertIfNotInClient() {
    alert('This button is unavailable as LIFF is currently being opened in an external browser.');
  }
  toggleAccessToken() {
    this.showAccessToken = !this.showAccessToken;
  }
  toggleProfileData() {
    this.showProfile = !this.showProfile;
  }
  toggleQrCodeReader() {
    this.showQrCodeResult = !this.showQrCodeResult;
  }
}
</script>
