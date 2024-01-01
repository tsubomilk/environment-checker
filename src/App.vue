<template>
  <div id="app">
    <DeviceInfo :osInfo="osInfo" :browserInfo="browserInfo" />
  </div>
</template>

<script>
import DeviceInfo from './components/DeviceInfo.vue';
import UAParser from 'ua-parser-js';

export default {
  name: 'App',
  components: {
    DeviceInfo
  },
  data() {
    return {
      osInfo: '',
      browserInfo: ''
    };
  },
  mounted() {
    this.getSoftwareInfo();
  },
  methods: {
    getSoftwareInfo() {
      const parser = new UAParser();
      const result = parser.getResult();

      this.osInfo = this.formatOSInfo(result.os);
      this.browserInfo = this.formatBrowserInfo(result.browser);
    },
    formatOSInfo(os) {
      let osInfo = `${os.name} ${os.version}`;
      if (os.name === 'Mac OS') {
        osInfo += ` (${this.getMacOSName(os.version)})`;
      }
      return osInfo;
    },
    formatBrowserInfo(browser) {
      return `${browser.name} ${browser.version}`;
    },
    getMacOSName(version) {
      const majorVersion = version.split('.')[1];
      switch (majorVersion) {
        case '15': return 'Catalina';
        case '14': return 'Mojave';
        case '13': return 'High Sierra';
        // 以下、必要に応じて他のバージョンを追加
        default: return 'Unknown';
      }
    }
  }
};
</script>




<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
