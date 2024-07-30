<template>
    <div class="parse">
      <h2>parse</h2>
      <input v-model="fileUrl" placeholder="enter the path of pdffile" class="address-input" />
      <button @click="parsepdf">starting to parse</button>
      <div class="filename">{{ fileUrl }}</div>
      <div class="resaddre">{{ `save at:${resaddr}` }}</div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'parse',
  data() {
    return {
      fileUrl: '',
      error: null,
      resaddr:'',
      parsered:false,
      responseTime: null,
    };
  },
  methods:{
    async parsepdf() {
    this.error = null;
    try {
      const response = await axios.post('http://127.0.0.1:5000/parse', {
        url: this.fileUrl,
        // respondemode:this.respondemode
      });
      this.parsered=response.data.res
      this.resaddr=response.data.where
      } catch (error) {
        this.error = `Error: ${error.message}`;
    }
    }
},
}
</script>

<style scoped>
.parse {
  position: fixed;
  top: 10px;
  left: 500px;
  width: 400px; /* 增加宽度以适应滚动条 */
  height: 95vh; /* 设置高度以使其可滚动 */
  padding: 20px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden; /* 隐藏默认溢出 */
}
button {
  padding: 5px 10px;
  margin-bottom: 20px; /* 增加按钮与对话之间的间隔 */
}
.filename{
  width: 100%;
  padding: 5px;
  margin-bottom: 10px;
}
.resaddre{
  width: 100%;
  padding: 5px;
  margin-bottom: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
input[type="file"] {
  display: block;
  margin-bottom: 10px;
}
</style>

