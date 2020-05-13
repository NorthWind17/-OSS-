<template>
    <div class="imgUpload" id="mpupload">
        <!-- <div id="ossfile">你的浏览器不支持flash,Silverlight或者HTML5！</div> -->

        <div id="container">
            <a id="selectfiles" href="javascript:void(0);" class="btn">
                <el-button type="primary" plain>选择文件</el-button>
            </a>
            <!-- <a id="postfiles" href="javascript:void(0);" class="btn">
                <el-button type="primary" plain @click="send">开始上传</el-button>
            </a>-->
        </div>
        <div>
            <span style="color:red;">*</span>可上传jpg,gif,png,bmp,jpeg文件
        </div>
        <ul id="ossfilea" class="ossfile" style="display:block">
            <li id="arr_li" v-for="(item,index) in licenceImg">
                <div class="tukuang">
                    <img
                        @click="lookimg(item)"
                        class="imgge"
                        style="width:86px;height: 60px;float:left"
                        :src="item"
                        alt="图片"
                    />
                    <div class="tuu" :id="newFiles[index].id">{{newFiles[index].name}}</div>
                    <div class="clear"></div>
                </div>
                <span class="dela" @click="deletePic(index)">×</span>
            </li>
        </ul>
    </div>
</template>

<script>
import * as dd from 'dingtalk-jsapi';
import axios from 'axios';
// import $ from 'jquery';
// import plupload from 'plupload';
//
var accessid = '';
var accesskey = '';
var host = '';
var policyBase64 = '';
var signature = '';
var callbackbody = '';
var filename = '';
var key = '';
var expire = 0;
var g_object_name = '';
var g_object_name_type = 'random_name';
var now = Date.parse(new Date()) / 1000;
var timestamp = Date.parse(new Date()) / 1000;
var hostU = '';
var hostUp = '';
var uploader;
// var upImgList=[];
//             var licenceImg=[];

function get_signature() {
    axios.post('/ossapi/sign/get').then(
        function(response) {
            hostUp = response.data.host;
            let obj = response.data;

            host = obj['host'];
            policyBase64 = obj['policy'];
            accessid = obj['accessid'];
            signature = obj['signature'];
            expire = parseInt(obj['expire']);
            callbackbody = obj['callback'];
            key = obj['dir'];
            return true;
        },
        function(err) {
            console.log(err);
        }
    );
}
//随机名字
function random_string(len) {
    var len = len || 32;
    var chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
    var maxPos = chars.length;
    var pwd = '';
    for (let i = 0; i < len; i++) {
        pwd += chars.charAt(Math.floor(Math.random() * maxPos));
    }
    return pwd + new Date().getTime();
}

function get_suffix(filename) {
    let pos = filename.lastIndexOf('.');
    let suffix = '';
    if (pos != -1) {
        suffix = filename.substring(pos);
    }
    return suffix;
}

//计算文件名
function calculate_object_name(filename) {
    if (g_object_name_type == 'local_name') {
        g_object_name += '${filename}';
    } else if (g_object_name_type == 'random_name') {
        let suffix = get_suffix(filename);
        g_object_name = key + random_string(10) + suffix;
        // hostU = g_object_name;
    }
    return '';
}

//得到上传文件名
function get_uploaded_object_name(filename) {
    if (g_object_name_type == 'local_name') {
        tmp_name = g_object_name;
        tmp_name = tmp_name.replace('${filename}', filename);
        return tmp_name;
    } else if (g_object_name_type == 'random_name') {
        return g_object_name;
    }
}

//设置上传参数
function set_upload_param(up, filename, ret) {
    if (ret == false) {
        ret = get_signature();
    }
    g_object_name = key;
    if (filename != '') {
        let suffix = get_suffix(filename);
        calculate_object_name(filename);
    }
    hostU = g_object_name;
    let new_multipart_params = {
        key: g_object_name,
        policy: policyBase64,
        OSSAccessKeyId: accessid,
        success_action_status: '200', //让服务端返回200,不然，默认会返回204
        callback: callbackbody,
        signature: signature
    };
    up.setOption({
        url: host,
        multipart_params: new_multipart_params
    });

    up.start();
}

export default {
    // model: {
    //     prop: 'msg',
    //     event: 'ee'
    // },
    props: {
        msg: '',
        licenceImg: {
            type: Array
        },
        upImgList: {
            type: Array
        },
        isShow: {
            type: Array
        }
    },
    data() {
        return {
            url: this.msg || '',
            newFiles: [],
            newUp: '',
            licenceImg: [],
            aaa: []
        };
    },
    created() {},
    mounted() {
        this.initPlUploader();
    },
    methods: {
        lookimg(item) {
            dd.ready(function() {
                dd.biz.util.previewImage({
                    urls: [item], //图片地址列表
                    current: item, //当前显示的图片链接
                    onSuccess: function(result) {
                        /**/
                    },
                    onFail: function(err) {}
                });
            });
        },
        /**
         * 初始化上传插件
         */
        initPlUploader() {
            const _this = this;
            uploader = new plupload.Uploader({
                runtimes: 'html5,flash,silverlight,html4',
                browse_button: 'selectfiles',
                //multi_selection: false,
                container: document.getElementById('container'),
                flash_swf_url: '../../../static/js/plupload-2.1.2/js/Moxie.swf',
                silverlight_xap_url:
                    '../../../static/js/plupload-2.1.2/js/Moxie.xap',
                url: 'https://oss.aliyuncs.com',

                filters: {
                    mime_types: [
                        //只允许上传图片和zip文件
                        {
                            title: 'Image files',
                            extensions: 'jpg,gif,png,bmp,jpeg'
                        }
                    ],
                    max_file_size: '20mb', //最大只能上传10mb的文件
                    prevent_duplicates: false //不允许选取重复文件
                },
                multi_selection: true,
                multiple_queues: true,
                init: {
                    PostInit: function() {
                        get_signature();
                        // document.getElementById('ossfile').innerHTML = '';
                        // document.getElementById(
                        //     'postfiles'
                        // ).onclick = function() {
                        //     set_upload_param(uploader, '', false);
                        //     return false;
                        // };
                    },

                    FilesAdded: function(up, files) {
                        if (uploader.files.length <= 9) {
                            plupload.each(files, function(file) {
                                if (uploader.files.length != 0) {
                                    set_upload_param(uploader, '', true);
                                    return false;
                                } else {
                                    _this.$alert('请添加图片');
                                }
                            });
                        } else {
                            _this.$alert('最多只能上传9张图片');
                            uploader.files.length = _this.licenceImg.length;
                            //uploader.files.splice(1, 1);

                            //'<div class="progress"><div class="progress-bar" style="width: 0%"></div></div>'
                        }
                    },

                    BeforeUpload: function(up, file) {
                        // check_object_radio();
                        set_upload_param(up, file.name, true);
                    },

                    // UploadProgress: function(up, file) {
                    //     var d = document.getElementById(file.id);
                    //     d.getElementsByTagName('b')[0].innerHTML =
                    //         '<span>' + file.percent + '%</span>';
                    //     var prog = d.getElementsByTagName('div')[0];
                    //     var progBar = prog.getElementsByTagName('div')[0];
                    //     progBar.style.width = 2 * file.percent + 'px';
                    //     progBar.setAttribute('aria-valuenow', file.percent);
                    // },

                    FileUploaded: function(up, file, info) {
                        if (info.status == 200) {
                            // let imageUrl = host + '/' + hostU;

                            _this.upImgList = [];
                            _this.newFiles.push(file);
                            _this.newUp = plupload;
                            _this.upImgList.push(
                                get_uploaded_object_name(file.name)
                            );
                            let len = _this.upImgList.length;

                            for (var b = 0; b < len; b++) {
                                _this.licenceImg.push(
                                    hostUp + '/' + _this.upImgList[b]
                                );
                            }

                            _this.$emit('listenToChildEvent', _this.licenceImg);

                            // document.getElementById('ossfilea').innerHTML += '';
                            // $('#ossfilea').html();
                            // 引入的图片
                            // $('#ossfilea').empty();
                            // for (var k = 0; k < len; k++) {
                            //     var li_img =
                            //         '<li class="arr_li"><div class="tukuang"><img class="imgge" style="width:86px;height: 60px;float:left" src="' +
                            //         _this.licenceImg[k] +
                            //         '" alt=""/><div class="tuu" id="' +
                            //         file.id +
                            //         '">' +
                            //         file.name +
                            //         ' (' +
                            //         plupload.formatSize(file.size) +
                            //         ')</div><div class="clear"></div></div><span class="dela" >×</span></li>';
                            //     $('#ossfilea').append(li_img);
                            // }
                        } else if (info.status == 203) {
                        } else {
                        }
                    },

                    Error: function(up, err) {
                        if (err.code == -600) {
                            _this.$message({
                                showClose: true,
                                message: '上传文件过大',
                                type: 'error'
                            });
                        } else if (err.code == -601) {
                            _this.$message({
                                showClose: true,
                                message:
                                    '暂不支持此类文件，请上传xls或者xlxs的格式文件',
                                type: 'error'
                            });
                        } else if (err.code == -602) {
                            _this.$message({
                                showClose: true,
                                message: '已经上传过一次',
                                type: 'error'
                            });
                        } else {
                            _this.$message({
                                showClose: true,
                                message: '上传错误',
                                type: 'error'
                            });
                        }
                    }
                }
            });
            uploader.init();
            _this.upImgList = [];
            // $('#ossfilea').on('click', 'li .dela', function() {
            // var i = $(this)
            //     .closest('li')
            //     .index();
            // $(this)
            //     .closest('li')
            //     .remove();
            // console.log('44++' + _this.licenceImg);
            // uploader.files.pop();
            // _this.upImgList.splice(i, 1);
            // _this.licenceImg.splice(i, 1);
            // console.log('66++' + _this.licenceImg);
            // _this.$emit('listenToChildEvent', _this.licenceImg);
            // });
        },
        deletePic(index) {
            // uploader.files.pop();
            uploader.files.splice(index, 1);
            this.newFiles.splice(index, 1);
            this.upImgList.splice(index, 1);
            this.licenceImg.splice(index, 1);
            this.$emit('listenToChildEvent', this.licenceImg);
        }
    },
    watch: {
        isShow: {
            handler(newValue, oldValue) {
                if (newValue.length === 0) {
                    document.getElementById('ossfilea').innerHTML = '';
                    uploader.files.length = 0;
                    this.newFiles.length = 0;
                }
            },
            deep: true
        }
    }
};
</script>
<style>
#ossfilea .tuu {
    float: left;
    width: 60%;
    line-height: 60px;
    margin-left: 20px;
    overflow: hidden;
    display: block;
    text-overflow: ellipsis;
    white-space: nowrap;
}
#mpupload #arr_li {
    position: relative;
    margin-bottom: 10px;
}
#mpupload .tukuang {
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 5px;
    display: block;
}
#mpupload .clear {
    clear: both;
}
#mpupload .dela {
    width: 16px;
    height: 16px;
    background: #e61b1b;
    display: block;
    text-align: center;
    line-height: 13px;
    color: #fff;
    font-size: 20px;
    border-radius: 50%;
    position: absolute;
    top: -5px;
    right: -6px;
    box-sizing: border-box;
    cursor: pointer;
}
</style>
