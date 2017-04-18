# vue-qiniu

> A Vue.js project

# 用法

input[type="file"] change事件触发后，先去（如果是图片，可以同时通过FileReader以及readAsDataURL将图片预览在页面上）后台请求七牛的上传token，将拿到的token和key以及通过change传递过来的files一起append到formData中。然后将formData通过post传递给七牛，七牛在处理后将返回真正的文件地址

获取TOKEN

const qiniu = require('qiniu')
const crypto = require('crypto')
const Config = require('qiniu-config')

exports.token = function*() {
    //构建一个保存文件名
    //这里没有处理文件后缀,需要自己传递过来,然后在这里处理加在key上,非必须
    const key = crypto.createHash('md5').update(((new Date()) * 1 + Math.floor(Math.random() * 10).toString())).digest('hex')
    //Config 七牛的秘钥等配置
    const [ACCESS_KEY, SECRET_KEY, BUCKET] = [Config.accessKey, Config.secretKey, Config.bucket] 
    qiniu.conf.ACCESS_KEY = ACCESS_KEY
    qiniu.conf.SECRET_KEY = SECRET_KEY
    const upToken = new qiniu.rs.PutPolicy(BUCKET + ":" + key)
    try {
        const token = upToken.token()
        return this.body = {
            key: key,
            token: token
        }
    } catch (e) {
        // throw error
    }
}

//假设api 地址是 /api/token
上传组件 UPLOAD.VUE

<template>
    <label class="mo-upload">
        <input type="file" :accept="accepts" @change="upload">
        <slot></slot>
    </label>
</template>
<style lang="scss">
    .mo-upload {
        display: inline-block;
        position: relative;
        margin-bottom: 0;
        input[type="file"] {
            display: none;
        }
        .mo-upload--label {
            display: inline-block;
            position: relative;
        }
    }
</style>
<script>
    export default {
        name : 'MoUpload',
        props : {
            accepts : { //允许的上传类型
                type : String,
                default : 'image/jpeg,image/jpg,image/png,image/gif'
            },
            flag : [String, Number], //当前上传标识,以便于在同一个监听函数中区分不同的上传域
            maxSize : {
                type : Number,
                default : 0 //上传大小限制
            }, 
        },
        methods: {
            upload (event) {
                let file = event.target.files[0]
                const self = this
                const flag = this.flag
                if (file) {
                    if (this.maxSize) {
                        //todo filter file
                    }
                    //filter file, 文件大小,类型等过滤
                    //如果是图片文件
                    // const reader = new FileReader()
                    // const imageUrl = reader.readAsDataURL(file)
                    // img.src = imageUrl //在预览区域插入图片

                    const formData = new FormData()
                    formData.append('file', file)

                    //获取token
                    this.$http.get(`/api/token/`)
                    .then(response => {
                        const result = response.body
                        formData.append('token', result.token)
                        formData.append('key', result.key)
                        //提交给七牛处理
                        self.$http.post('https://up.qbox.me/', formData, {
                            progress(event) {
                                //传递给父组件的progress方法
                                self.$emit('progress', parseFloat(event.loaded / event.total * 100), flag) 
                            }
                        })
                        .then(response => {
                            const result = response.body
                            if (result.hash && result.key) {
                                //传递给父组件的complete方法
                                self.$emit('complete', 200 , result, flag)
                                //让当前target可以重新选择
                                event.target.value = null
                            } else {
                                self.$emit('complete', 500, result, flag)
                            }
                        }, error => self.$emit('complete', 500, error.message), flag)
                    })
                }
            }
        }
    }
</script>
父组件调用

<template>
    <section>
        ...
        <figure class="upload-preview">
            <img :src="thumbnail" v-if="thumbnail"/>
        </figure>
        <mo-upload flag="'thumbnail'" @complete="uploadComplete" @progress="uploadProgress">
            <a>选择图片文件<i class="progress" :style="{width:progress + '%'}"></i></a>
        </mo-upload>
        ...
    </section>
</template>
<script>
    import MoUpload from 'upload'
    export default {
        components : {
            MoUpload,
        },
        data () {
            return {
                thumbnail : null,
                progress : 0 //上传进度
            }
        },
        methods : {
            uploadProgress (progress, flag) {
                //这里可以通过回调的flag对不同上传域做处理
                this.progress = progress < 100 ? progress : 0;
            },
            uploadComplete(status, result, flag) {
                if (status == 200) { //
                    this.thumbnail = `domain.com/${result.key}` //七牛域名 + 返回的key 组成文件url
                } else {
                    //失败处理
                }
            },
        }
    }
</script>
小结

相比于FILEApi 或者其他上传组件，这种方法代码量最小。但是缺点也是显而易见的，大量html5 API的使用，势必会回到兼容性这个老大难上来，慎重的选择性使用吧

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

