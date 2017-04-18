<template>
  <div id="app">
    <div style="background-color: #fff; height: 100vh;">
    </div>
    <div style="position: relative; z-index: 9999; background-color: #fff;">
      <h1 style="margin:0;">Upload</h1>
      <figure class="upload-preview">
            <img :src="thumbnail" v-if="thumbnail"/>
        </figure>
        <mo-upload flag="'thumbnail'" @complete="uploadComplete" @progress="uploadProgress">
            <a>选择图片文件<i class="progress" :style="{width:progress + '%'}"></i></a>
        </mo-upload>
    </div>
  </div>
</template>
<script>
import Upload from './components/upload'

export default {
  name: 'app',
  components: {
    Upload
  }
}
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: relative;
}
</style>
