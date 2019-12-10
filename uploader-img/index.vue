<template>
    <div class="uploader-file-box">
        <!--图片显示和操作-->
        <div class="h-uploader h-uploader-images-container">
            <!--多图-->
            <div v-if="imgArr.length > 0">
                <div class="h-uploader-image uploader-image-empty" v-for="(item, index) in imgArr" :key="index">
                    <div class="h-uploader-image-background" :style="{'background-image': 'url(' + item + ')'}"></div>
                    <div class="h-uploader-image-operate">
                        <div>
                            <span class="h-uploader-operate" @click="openPreview(index)"><i class="h-icon-fullscreen"/></span>
                            <i class="h-split" style="width: 3px;"/>
                            <span class="h-uploader-operate" @click="trashImage(index)"><i class="h-icon-trash"/></span>
                        </div>
                    </div>
                </div>
            </div>
            <!--单图-->
            <div v-if="imgUrl">
                <div class="h-uploader-image uploader-image-empty">
                    <div class="h-uploader-image-background" :style="{'background-image': 'url(' + imgUrl + ')'}"></div>
                    <div class="h-uploader-image-operate">
                        <div>
                            <span class="h-uploader-operate" @click="openPreview(imgUrl)"><i class="h-icon-fullscreen"/></span>
                            <i class="h-split" style="width: 3px;"/>
                            <span class="h-uploader-operate" @click="trashImage"><i class="h-icon-trash"/></span>
                        </div>
                    </div>
                </div>
            </div>
            <!--进度条-->
            <div class="h-uploader-image-empty h-uploader-browse-button progress-box" v-if="precentShow">
                <p>上传中..</p>
                <Progress :percent="precent" class="progress"/>
            </div>
            <!--添加-->
            <div class="h-uploader-image-empty h-uploader-browse-button" @click="AddUpBtn" v-if="AddPlus">
                <i class="h-icon-plus"/>
            </div>
        </div>
        <input type="file" ref="file" id="file" style="display: none;" @change="GetFile($event)" :accept="option.accept">
        <!--裁剪图片-->
        <Modal v-model="OpenModal" hasCloseIcon hasDivider :closeOnMask="false" middle>
            <header class="h-modal-header">裁剪图片</header>
            <div class="cropper-box">
                <vueCropper ref="cropper" :img="option.img" autoCrop :fixed="option.fixed" :fixedNumber="option.fixedNumber"/>
            </div>
            <footer class="h-modal-footer">
                <button class="h-btn h-btn-primary" @click="YesModal" :loading="isLoading">确定上传</button>
                <button class="h-btn" @click="OpenModal = false">取消关闭</button>
            </footer>
        </Modal>
    </div>
</template>

<script>
    import { VueCropper }  from 'vue-cropper';
    import { UploadFile } from '@/utils/request';
    export default {
        name: "UploaderImg",
        components: {VueCropper},
        data() {
            return {
                option: {img: ''}, imgArr: [], imgUrl:'', OpenModal: false, isLoading: false,  fileName: '', imgType: 0, AddPlus: true,
                precent: 0, precentShow: false,
            }
        },
        props: {
            img: {
                type: String,
                default: ''
            },
            img_arr: {
                type: Array,
                default() {
                    return [];
                }
            },
            config: {
                type: Object,
                default() {
                    return {};
                }
            }
        },
        mounted() {
            //默认图片地址配置
            this.imgUrl = this.img;
            this.imgArr = this.img_arr;
            if (this.img && this.img_arr.length <= 0) {
                this.imgType = 0;   //字符串
            } else if (!this.img && this.img_arr.length >= 1) {
                this.imgType = 1;   //数组
            } else {
                this.imgType = 1;   //数组
            }
            //裁剪器配置
            this.option.accept = this.config.accept ? this.config.accept : 'image/png, image/jpeg, image/gif, image/jpg';
            this.option.outputType = this.config.outputType ? this.config.outputType : 'jpeg';
            this.option.fixedNumber = this.config.fixedNumber ? this.config.fixedNumber : [];
            this.option.canMoveBox = this.config.canMoveBox ? this.config.canMoveBox : true;
            this.option.fileName = this.config.fileName ? this.config.fileName : 'file';
            this.option.original = this.config.original ? this.config.original : false;
            this.option.fixedBox = this.config.fixedBox ? this.config.fixedBox : false;
            this.option.canScale = this.config.canScale ? this.config.canScale : true;
            this.option.canMove = this.config.canMove ? this.config.canMove : true;
            this.option.fileNum = this.config.fileNum ? this.config.fileNum : 1;
            this.option.fixed = this.config.fixed ? this.config.fixed : false;
            this.option.full = this.config.full ? this.config.full : false;
            this.option.info = this.config.info ? this.config.info : true;
            //初始化
            this.initAddPlus();
        },
        watch: {
            //动态更新
            img(url) {
                this.imgUrl = url;
            },
            img_arr(arr) {
                this.imgArr = arr;
            }
        },
        methods: {
            initAddPlus() {
                let img_arr = this.img_arr.length;
                let fileNum = this.option.fileNum;
                if (this.imgType === 0 && this.imgUrl) {
                    this.AddPlus = false;
                } else if (this.imgType === 0 && !this.imgUrl) {
                    this.AddPlus = true;
                } else if (this.imgType === 1 && img_arr < fileNum) {
                    this.AddPlus = true;
                } else if (this.imgType === 1 && img_arr >= fileNum) {
                    this.AddPlus = false;
                } else if (this.imgType === 1 && fileNum === 0) {
                    this.AddPlus = true;
                } else {
                    this.AddPlus = true;
                }
            },
            //选择文件
            AddUpBtn() {
                //单图
                if (this.imgType === 0) {
                    if (this.imgUrl === '') {
                        this.$refs.file.dispatchEvent(new MouseEvent('click'));
                    } else {
                        this.$Message.error('当前模式只能上传1张图片');
                    }
                }
                //多图
                if (this.imgType === 1) {
                    let img_arr = this.img_arr.length;
                    let fileNum = this.option.fileNum;
                    if (fileNum === 0 || img_arr < fileNum) {
                        this.$refs.file.dispatchEvent(new MouseEvent('click'));
                    } else {
                        this.$Message.error('最多只能上传' + this.option.fileNum + '张图片');
                    }
                }
            },
            //文件选择完成事件
            GetFile(e) {
                let file = e.target.files[0];
                this.fileName = file.name;
                let reader = new FileReader();
                reader.onload = (e) => {
                    this.option.img = window.URL.createObjectURL(
                        new Blob(
                            [e.target.result]
                        )
                    );
                };
                reader.readAsArrayBuffer(file);
                this.OpenModal = true;
            },
            //裁剪完成，上传文件
            YesModal() {
                let param = new FormData();
                let that = this; that.isLoading = true;
                //获取裁剪后的文件
                this.$refs.cropper.getCropBlob((data) => {
                    param.append(this.option.fileName, data, this.fileName);
                    that.precentShow = true;
                    //开始上传
                    UploadFile(param,(res) => {
                        let loaded = res.loaded, total = res.total;
                        let precent = (loaded/total) * 100;
                        that.precent = precent;
                        //上传完成
                        if (precent >= 100) {
                            that.precentShow = false;
                        }
                    }).then(res => {
                        that.isLoading = false;
                        if (res.code === 1000) {
                            //单图
                            if (that.imgType === 0) {
                                that.imgUrl = res.data.url;
                            }
                            //多图
                            if (that.imgType === 1) {
                                that.imgArr.push(res.data.url);
                            }
                            that.$Message.success(res.msg);
                            that.OpenModal = false;
                            that.initAddPlus();
                        } else {
                            that.$Message.error(res.msg);
                        }
                    });
                });
            },
            //图片预览
            openPreview(url) {
                //单图
                if (this.imgType === 0) {
                    this.$ImagePreview(url, 0);
                }
                //多图
                if (this.imgType === 1) {
                    this.$ImagePreview(this.imgArr[url], 0);
                }
            },
            //删除选中图片
            trashImage(index) {
                //单图
                if (this.imgType === 0) {
                    this.imgUrl = '';
                }
                //多图
                if (this.imgType === 1) {
                    this.imgArr.splice(index,1);
                }
                this.initAddPlus();
            }
        }
    }
</script>

<style scoped>
    .cropper-box {
        padding: 15px 0px;
        height: 400px;
        width: 400px;
    }
    .uploader-file-box {
        margin-bottom: 24px;
    }
    .h-uploader-image, .h-uploader-image-empty {
        height: 100px;
        width: 100px;
    }
    .progress-box {
        margin-left: 10px;
    }
    .progress-box p{
        text-align: center;
        margin-top: 26px;
        font-size: 14px;
    }
    .progress-box .progress{
        margin: 0 5px;
    }
    .uploader-image-empty {
        border: 1px dashed #c1c1c1;
    }
    .uploader-image-empty:hover{
         border: 1px dashed #409eff;
    }
</style>
