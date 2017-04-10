<template>
    <label class="mo-upload">
        <input type="file" :accept="accepts" @change="upload">
        <slot></slot>
    </label>
</template>
<style>
.mo-upload {
    display: inline-block;
    position: relative;
    margin-bottom: 0;
}
.mo-upload input[type="file"]{
    display: none;
}

.mo-upload  .mo-upload--label {
    display: inline-block;
    position: relative;
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
            saveKey: '' 
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
                    let saveKey = this.saveKey
                    if(!saveKey){
                        saveKey = file.name
                    }else{
                        let [name, ext] = file.name.split('.')
                        saveKey = saveKey + '.' + ext
                    }
                    //filter file, 文件大小,类型等过滤
                    //如果是图片文件
                    // const reader = new FileReader()
                    // const imageUrl = reader.readAsDataURL(file)
                    // img.src = imageUrl //在预览区域插入图片

                    const formData = new FormData()
                    formData.append('file', file)
                    
                    //获取token
                    this.$http.get('/api/users/me/uploadToken', { params:{key: saveKey}})
                    .then(response => {
                        const result = response.body
                        formData.append('token', result.uptoken)
                        formData.append('key', result.key)
                        // domain on5wi69cb.bkt.clouddn.com
                        //提交给七牛处理
                        self.$http.post('https://up.qbox.me/', formData, {
                            credentials:false,
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