<template>
  <div>
    <h1>デバイス情報</h1>
    <h3>ソフトウェア</h3>
    <p>OS: {{ osInfo }}</p>
    <p>{{ browserInfo }}</p>
    <h3>ハードウェア</h3>
    <p>{{ cpuInfo }}</p>
    <p>{{ gpuInfo }}</p>
    <p>{{ memoryInfo }}</p>
    <p>{{ batteryInfo }}</p>
    <h3>ネットワーク</h3>
    <p>{{ publicIP }}</p>
    <p>{{ serviceProvider }}</p>
    <p>{{ downloadSpeed }}</p>
  </div>
</template>

<script>
export default {
  props: {
    osInfo: String,
    browserInfo: String
  },
  data() {
    return {
      cpuInfo: 'CPU情報を取得中...',
      gpuInfo: 'GPU情報を取得中...',
      memoryInfo: 'メモリ情報を取得中...',
      batteryInfo: 'バッテリー情報を取得中...',
      publicIP: '公開IPを取得中...',
      serviceProvider: 'サービスプロバイダを取得中...',
      downloadSpeed: 'ダウンロード速度を計測中...'
    };
  },
  mounted() {
    this.getCPUInfo();
    this.getGPUInfo();
    this.getMemoryInfo();
    this.getBatteryInfo();
    this.getPublicIP();
    this.measureDownloadSpeed();
  },
  methods: {
    getCPUInfo() {
      this.cpuInfo = `CPU Cores: ${navigator.hardwareConcurrency}`;
    },
    getGPUInfo() {
      const canvas = document.createElement('canvas');
      const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
      if (!gl) {
        this.gpuInfo = 'WebGL not supported';
        return;
      }
      const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
      this.gpuInfo = debugInfo 
        ? `GPU: ${gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL)} / ${gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL)}`
        : 'Cannot determine GPU info';
    },
    getMemoryInfo() {
      if (performance.memory) {
        this.memoryInfo = `メモリ情報: Limit: ${Math.round(performance.memory.jsHeapSizeLimit / 1048576)}MB, ` +
          `Total: ${Math.round(performance.memory.totalJSHeapSize / 1048576)}MB, ` +
          `Used: ${Math.round(performance.memory.usedJSHeapSize / 1048576)}MB`;
      } else {
        this.memoryInfo = 'メモリ情報は利用できません';
      }
    },
    getBatteryInfo() {
      navigator.getBattery().then(battery => {
        this.batteryInfo = `バッテリー: Charging: ${battery.charging ? 'charging' : 'not charging'}, ` +
          `Level: ${Math.round(battery.level * 100)}%`;
      });
    },
    getPublicIP() {
      fetch('http://ip-api.com/json/')
        .then(response => response.json())
        .then(data => {
          this.publicIP = `Public IP: ${data.query}`;
          this.serviceProvider = `Service Provider: ${data.isp}`;
        })
        .catch(error => {
          console.error('Error fetching IP information:', error);
        });
    },
    measureDownloadSpeed() {
      const startTime = (new Date()).getTime();
      const fileUrl = 'https://drive.google.com/u/4/uc?id=1EEpAbOJkOdtaeOOrlgJ1d1CJwHh_vv6Y&export=download' //大きなファイルのURL

      fetch(fileUrl)
        .then(response => {
          const endTime = (new Date()).getTime();
          const duration = (endTime - startTime) / 1000; // 秒単位
          const fileSize = response.headers.get('Content-Length'); // バイト単位
          const speed = fileSize / duration / 1024 / 1024; // Mbps

          this.downloadSpeed = `Download Speed: ${speed.toFixed(2)} Mbps`;
        })
        .catch(error => {
          console.error('Error measuring download speed:', error);
        });
    }
  }
};
</script>

<style scoped>
/* ここにスタイルを追加 */
</style>
