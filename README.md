# VUE 图片裁剪上传组件 v1.0

此组件是使用HeyUI的UI框架结合vue-cropper图片裁剪库，做的一个图片裁剪/上传一体的组件。

######下列示例图仅供参考，真实效果可能会有些许的差别。

![](https://cdn.nlark.com/yuque/0/2019/png/285274/1575967666100-assets/web-upload/36163063-63ac-484e-88c6-f4ea9d68f6a7.png)
![](https://cdn.nlark.com/yuque/0/2019/png/285274/1575967666102-assets/web-upload/b42b65d7-907c-4600-ba38-1f219f5ac7d6.png)

![](https://cdn.nlark.com/yuque/0/2019/png/285274/1575967666257-assets/web-upload/769d4d7f-7369-41ce-908c-31ca038974a3.png)

<hr>

## 需要的依赖库
```
vue-cropper
```

地址：[vue-router](https://github.com/vuejs/vue-router "vue-router")

<hr>

### 基于UI框架
```
heyui
```

地址：[heyui](https://www.heyui.top/ "heyui")

<hr>

### 使用方法
```
import UploaderImg  from '@/views/common/uploader-img';

components: { 
    UploaderImg
}

<UploaderImg :img="img_url" :config="UpImgConfig"/>
```

<hr>

### 参数说明
| 参数 | 参数说明 | 类型 | 默认值 |
| ---- | ---- | ---- | ---- |
| img | 字符串类型的图片地址（单图） | String (字符串) | 空 |
| img_arr | 数组型的图片地址（多图） | Array (数组) | 空 |
| config | 相关配置 | Object (对象) | 详见配置说明 |

#####img和img_arr 任传一个即可，不需要全部都赋值

<hr>

### 配置说明
| 参数 | 参数说明 | 类型 | 默认值 | 可选值 |
| ---- | ---- | ---- | ---- | ---- |
| accept | 文件选择器筛选 | String | image/png, image/jpeg, image/gif, image/jpg | 自行参考 |
| outputType | 裁剪生成图片的格式 (jpg 需要传入jpeg) | String | jpeg | jpeg / png / webp |
| info | 裁剪框的大小信息 | Boolean | true | true / false |
| canScale | 图片是否允许滚轮缩放 | Boolean |  true | true / false |
| fixed | 是否开启截图框宽高固定比例 | Boolean | false | true / false |
| fixedNumber | 截图框的宽高比例 | Array |  [1, 1] | [宽度, 高度] |
| full | 是否输出原图比例的截图 | Boolean | false | true / false |
| fixedBox | 固定截图框大小 不允许改变 | Boolean | false | true / false |
| canMove | 图片是否可以移动 | Boolean | true | true / false |
| canMoveBox | 截图框能否拖动 | Boolean | true | true / false |
| original | 图片按照原始比例渲染 | Boolean | false | true / false |
| fileNum | 限制上传张数（0为不限制） | int | 1 | 0 - N |
| fileName | 文件名称 | String | file | 省略（可无视此参数） |

<hr>

### 上传接口处理，并获得进度显示的处理代码
```
import axios from 'axios';

//上传图片
export const UploadFile= (file,callback) => {
    return new Promise((resolve, reject) => {
        Axio({
            url: uploadURL, //接口地址
            method:'post',
            data: file,
            //获取上传进度
            onUploadProgress: function(progressEvent){
                if(progressEvent.lengthComputable) {
                    //返回上传进度
                    callback(progressEvent);
                }
            }
        }).then((res) => {
            //上传成功
            resolve(res.data);
        })
        .catch((error) => {
            //上传失败
            reject(error);
        });
    });
};

```

<hr>

### 组件里需要修改的地方
```
//第 58 行 上传请求接口，根据你自己的业务逻辑，进行更改
import { UploadFile } from '@/utils/request';  

//第 186 行 调用上传请求接口，根据你自己的业务逻辑，进行更改

UploadFile(param,(res) => {
    //上传进度
    let loaded = res.loaded, total = res.total;
    let precent = (loaded/total) * 100;
    that.precent = precent;
    if (precent >= 100) { //上传完成
        that.precentShow = false;
    }
}).then(res => {
    //上传完成
    that.isLoading = false;
    //状态码自行处理更改
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

```

<hr>

#####如有vue项目的需要找兼职的，可联系我，我可兼职搞哦~

#####邮箱：806606688@qq.com

<hr>
