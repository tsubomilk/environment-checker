<template>
    <div id="content-to-print">
      <h1>Device Info</h1>
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
    <button @click="exportToPDF">PDFとして保存</button>
</template>


<script>
import html2pdf from 'html2pdf.js';

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
        this.memoryInfo = 'メモリ情報はこのデバイスでは利用できません';
      }
    },
    getBatteryInfo() {
      if ('getBattery' in navigator) {
        // getBatteryが利用可能な場合、バッテリー情報を取得
        navigator.getBattery().then(battery => {
          let chargingStatus = battery.charging ? 'charging' : 'not charging';
          let level = Math.round(battery.level * 100);
          let dischargingTime = battery.dischargingTime;

          let timeRemaining = '';
          if (dischargingTime === Infinity) {
            timeRemaining = 'Calculating time remaining...';
          } else if (dischargingTime < 0) {
            timeRemaining = 'Unavailable';
          } else {
            let hours = Math.floor(dischargingTime / 3600);
            let minutes = Math.floor((dischargingTime % 3600) / 60);
            timeRemaining = `${hours}h ${minutes}m`;
          }

          this.batteryInfo = `Battery: ${chargingStatus}, Level: ${level}%, Time remaining: ${timeRemaining}`;
        });
      } else {
        // getBatteryが利用できない場合、メッセージを表示
        this.batteryInfo = 'バッテリー情報はこのデバイスでは利用できません';
      }
    },
    getPublicIP() {
      fetch('https://ipinfo.io/json')
        .then(response => response.json())
        .then(data => {
          this.publicIP = `Public IP: ${data.ip}`;
          this.serviceProvider = `Service Provider: ${data.org}`;
        })
        .catch(error => {
          console.error('Error fetching IP information:', error);
        });
    },
    measureDownloadSpeed() {
      const startTime = (new Date()).getTime();
      const fileUrl = 'https://sabnzbd.org/tests/internetspeed/50MB.bin'; // 大きなファイルのURL

      fetch(fileUrl)
        .then(response => {
          const endTime = (new Date()).getTime();
          const duration = (endTime - startTime) / 1000; // 秒単位
          const fileSize = response.headers.get('Content-Length'); // バイト単位
          if (!fileSize) {
            throw new Error('File size not available');
          }
          const speed = fileSize / duration / 1024 / 1024; // Mbps

          this.downloadSpeed = `Download Speed: ${speed.toFixed(2)} Mbps`;
        })
        .catch(error => {
          console.error('Error measuring download speed:', error);
          this.downloadSpeed = 'Unable to measure download speed';
        });
    },
    exportToPDF() {
      // PDFに含めたいコンテンツがある要素を取得
      const element = document.getElementById('content-to-print');

      // html2pdfオプション
      const options = {
        margin: [10, 10, 10, 20], // 上、右、下、左の余白（mm単位）
        filename: 'mypage.pdf',
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
      };

      // html2pdfを使用してPDFを作成しダウンロード
      html2pdf().from(element).set(options).save();
    }
  }
};
</script>

<style scoped>
button {
  background-color: #4CAF50; /* 緑色の背景 */
  color: white; /* テキストは白色 */
  padding: 15px 32px; /* パディング */
  text-align: center; /* テキストを中央揃え */
  text-decoration: none; /* テキストの下線をなくす */
  display: inline-block; /* インラインブロック要素として表示 */
  font-size: 16px; /* フォントサイズ */
  margin: 4px 2px; /* マージン */
  cursor: pointer; /* カーソルをポインターに */
  border: none; /* 境界線なし */
  border-radius: 5px; /* 角を丸く */
}

button:hover {
  background-color: #45a049; /* ホバー時の背景色を少し濃く */
}
</style>
