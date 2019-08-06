<template>
    <div class="imgUpload" id="mpupload">

        <div id="container">
            <a id="selectfiles" href="javascript:void(0);" class="btn">
                <el-button type="primary" plain>选择文件</el-button>
            </a>
        </div>
        <ul id="ossfilea" class="ossfile" style="display:block">
            
        </ul>
    </div>
</template>

<script>
import axios from 'axios';
import $ from 'jquery';
// import plupload from 'plupload';

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
// var upImgList=[];
//             var licenceImg=[];

function get_signature() {
    axios.post('/ossapi/sign/get').then(
        function(response) {
            console.log('1111+' + response);
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
get_signature();
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
    model: {
        prop: 'msg',
        event: 'ee'
    },
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

            pxurl: []
        };
    },
    created() {},
    mounted() {
        this.initPlUploader();
    },
    methods: {
        /**
         * 初始化上传插件
         */
        initPlUploader() {
            var _this = this;
            var uploader = new plupload.Uploader({
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
                        { title: 'Image files', extensions: 'jpg,gif,png,bmp' },
                        { title: 'Zip files', extensions: 'zip,rar,ipa' }
                    ],
                    max_file_size: '20mb', //最大只能上传10mb的文件
                    prevent_duplicates: true //不允许选取重复文件
                },

                init: {
                    PostInit: function() {
                        // document.getElementById('ossfile').innerHTML = '';
                        // document.getElementById(
                        //     'postfiles'
                        // ).onclick = function() {
                        //     set_upload_param(uploader, '', false);
                        //     return false;
                        // };
                    },

                    FilesAdded: function(up, files) {
                        if (uploader.files.length <= 5) {
                            console.log(files);
                            plupload.each(files, function(file) {
                                console.log(file);
                                // $scope.slider_name = file.name;
                                // console.log($scope.slider_name);
                                if (uploader.files.length != 0) {
                                    console.log(uploader);
                                    console.log(uploader.files);
                                    set_upload_param(uploader, '', true);
                                    return false;
                                } else {
                                    alert('请添加图片');
                                }
                            });
                        } else {
                            alert('最多只能上传5张图片');
                            //uploader.files.splice(1, 1);
                            console.log(uploader.files);
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
                        // var pxurl = [];
                        if (info.status == 200) {
                            // let imageUrl = host + '/' + hostU;
                            _this.licenceImg = [];//清空，防止多次添加数组错误
                            console.log(
                                '我+++' + get_uploaded_object_name(file.name)
                            );
                            _this.upImgList.push(
                                get_uploaded_object_name(file.name)
                            );

                            // pxurl.push(imageUrl);
                            // console.log(pxurl);
                            let len = _this.upImgList.length;
                            console.log('length66++' + _this.upImgList);
                            for (var b = 0; b < len; b++) {
                                _this.licenceImg.push(
                                    hostUp + '/' + _this.upImgList[b]
                                );
                            }
                            console.log('uplength++' + _this.upImgList.length);
                            console.log(
                                'lllllength++' + _this.licenceImg.length
                            );

                            _this.$emit('listenToChildEvent', _this.licenceImg);

                            console.log('length++' + _this.licenceImg);
                            document.getElementById('ossfilea').innerHTML += '';
                            $('#ossfilea').html();
                            // 引入的图片
                            $('#ossfilea').empty();
                            for (var k = 0; k < len; k++) {
                                var li_img =
                                    '<li class="arr_li"><div class="tukuang"><img class="imgge" style="width:86px;height: 60px;float:left" src="' +
                                    _this.licenceImg[k] +
                                    '" alt=""/><div class="tuu" id="' +
                                    file.id +
                                    '">' +
                                    file.name +
                                    ' (' +
                                    plupload.formatSize(file.size) +
                                    ')</div><div class="clear"></div></div><span class="dela" >×</span></li>';
                                $('#ossfilea').append(li_img);
                            }
                        } else if (info.status == 203) {
                        } else {
                        }
                    },

                    Error: function(up, err) {
                        if (err.code == -600) {
                            document
                                .getElementById('console')
                                .appendChild(
                                    document.createTextNode(
                                        '\n选择的文件太大了,可以根据应用情况，在upload.js 设置一下上传的最大大小'
                                    )
                                );
                        } else if (err.code == -601) {
                            document
                                .getElementById('console')
                                .appendChild(
                                    document.createTextNode(
                                        '\n选择的文件后缀不对,可以根据应用情况，在upload.js进行设置可允许的上传文件类型'
                                    )
                                );
                        } else if (err.code == -602) {
                            document
                                .getElementById('console')
                                .appendChild(
                                    document.createTextNode(
                                        '\n这个文件已经上传过一遍了'
                                    )
                                );
                        } else {
                            document
                                .getElementById('console')
                                .appendChild(
                                    document.createTextNode(
                                        '\nError xml:' + err.response
                                    )
                                );
                        }
                    }
                }
            });
            uploader.init();

            this.upImgList = [];
            $('#ossfilea').on('click', 'li .dela', function() {
                var i = $(this)
                    .closest('li')
                    .index();
                $(this)
                    .closest('li')
                    .remove();
                uploader.files.pop();
                _this.upImgList.splice(i, 1);
                console.log(_this.upImgList);

                _this.licenceImg.splice(i, 1);
                console.log(_this.licenceImg);
                _this.$emit('listenToChildEvent', _this.licenceImg);
            });
        }
    },
    watch: {
        isShow: {
            handler(newValue, oldValue) {
                if (newValue.length === 0) {
                    document.getElementById('ossfilea').innerHTML += '';
                    $('#ossfilea').html();
                    $('#ossfilea').empty();
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
#mpupload .arr_li {
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
