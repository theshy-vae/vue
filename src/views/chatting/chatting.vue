<template class="container">
    <div class="columns" style="padding: 10px; margin-bottom: 50px">
        <div class="column is-6 is-offset-3">
            <div style="background-color: white;
                   border-radius: 5px; box-shadow: 0 0 10px #ccc">
                <div  style="text-align: center; line-height: 50px;font-weight: bolder">
                    {{ chatUser }}
                </div>
                <div id="window" style="height: 350px; overflow:auto; border-top: 1px solid #ccc" v-html="content"></div>
                <div style="height: 200px"><textarea v-model="text" style="height: 160px; width: 100%; padding: 20px; border: none; border-top: 1px solid #ccc;
                                  border-bottom: 1px solid #ccc; outline: none"></textarea>
                                          <div style="text-align: right; padding-right: 10px">
                                              <el-button type="primary" size="mini" @click="send">发送</el-button>
                                          </div>
                                      </div>
            </div>
        </div>
        <div>
            <audio id="music" src="../../assets/music.mp3"></audio>
        </div>
    </div>
</template>

<script>

    let socket;
    var music;

    export default {
        name: "Im",
        data() {
            return {
                circleUrl: 'https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png',
                user: {},
                isCollapse: false,
                users: [],
                chatUser: '',
                text: "",
                messages: [],
                content: '',
                avatar:''   //聊天对方的头像
            }
        },
        created() {
            this.init()
        },
        methods: {
            send() {
                if (!this.text) {
                    this.$message({type: 'warning', message: "请输入内容"})
                } else {
                    if (typeof (WebSocket) == "undefined") {
                        console.log("您的浏览器不支持WebSocket");
                    } else {
                        console.log("您的浏览器支持WebSocket");
                        // 组装待发送的消息 json
                        // {"from": "zhang", "to": "admin", "text": "聊天文本"}
                        let message = {from: this.user.username, to: this.chatUser, text: this.text}
                        console.log(message)
                        socket.send(JSON.stringify(message));  // 将组装好的json发送给服务端，由服务端进行转发
                        this.messages.push({user: this.user.username, text: this.text})
                        // 构建消息内容，本人消息
                        this.createContent(null, this.user.username, this.text)
                        this.text = '';
                    }
                }
            },
            createContent(remoteUser, nowUser, text) {  // 这个方法是用来将 json的聊天消息数据转换成 html的。
                let html
                // 当前用户消息
                if (nowUser) { // nowUser 表示是否显示当前用户发送的聊天消息，绿色气泡
                    html = "<div ref='list' class=\"el-row\" style=\"padding: 5px 0\">\n" +
                        "  <div class=\"el-col el-col-22\" style=\"text-align: right; padding-right: 10px\">\n" +
                        "    <div class=\"tip left\">" + text + "</div>\n" +
                        "  </div>\n" +
                        "  <div class=\"el-col el-col-2\">\n" +
                        "  <span class=\"el-avatar el-avatar--circle\" style=\"height: 40px; width: 40px; line-height: 40px;\">\n" +
                        "    <img src="+this.user.avatar+" style=\"object-fit: cover;\">\n" +
                        "  </span>\n" +
                        "  </div>\n" +
                        "</div>";
                } else if (remoteUser) {   // remoteUser表示远程用户聊天消息，蓝色的气泡
                    html = "<div class=\"el-row\" style=\"padding: 5px 0\">\n" +
                        "  <div class=\"el-col el-col-2\" style=\"text-align: right\">\n" +
                        "  <span class=\"el-avatar el-avatar--circle\" style=\"height: 40px; width: 40px; line-height: 40px;\">\n" +
                        "    <img src="+this.avatar+" style=\"object-fit: cover;\">\n" +
                        "  </span>\n" +
                        "  </div>\n" +
                        "  <div class=\"el-col el-col-22\" style=\"text-align: left; padding-left: 10px\">\n" +
                        "    <div class=\"tip right\">" + text + "</div>\n" +
                        "  </div>\n" +
                        "</div>";
                }
                console.log(html)
                this.content += html;
                sessionStorage.setItem(this.chatUser,this.content);
                //发送或接收消息后滚动条到底部
                this.$nextTick(() => {
                    let msg = document.getElementById('window') // 获取对象
                    msg.scrollTop = msg.scrollHeight // 滚动高度
                })
            },
            init() {
                this.chatUser=this.$route.query.name
                this.avatar=this.$route.query.avatar
                if(sessionStorage.getItem(this.chatUser)!=null) this.content=sessionStorage.getItem(this.chatUser)
                this.user=this.$store.getters.user
                let username = this.user.username;
                let _this = this;
                if (typeof (WebSocket) == "undefined") {
                    console.log("您的浏览器不支持WebSocket");
                } else {
                    console.log("您的浏览器支持WebSocket");
                    let socketUrl = process.env.VUE_APP_SOCKET_URL + this.$store.getters.user.username;
                    if (socket != null) {
                        socket.close();
                        socket = null;
                    }
                    // 开启一个websocket服务
                    socket=new WebSocket(socketUrl)
                    //打开事件
                    socket.onopen = function () {
                        console.log("websocket已打开");
                    };
                    //  浏览器端收消息，获得从服务端发送过来的文本消息
                    socket.onmessage = function (msg) {
                        console.log("收到数据====" + msg.data)
                        let data = JSON.parse(msg.data)  // 对收到的json数据进行解析， 类似这样的： {"users": [{"username": "zhang"},{ "username": "admin"}]}
                        if (data.users) {  // 获取在线人员信息
                            _this.users = data.users.filter(user => user.username !== username)  // 获取当前连接的所有用户信息，并且排除自身，自己不会出现在自己的聊天列表里
                        } else {
                            document.getElementById('music').play()
                            // 如果服务器端发送过来的json数据 不包含 users 这个key，那么发送过来的就是聊天文本json数据
                            //  // {"from": "zhang", "text": "hello"}
                            if (data.from === _this.chatUser) {
                                _this.messages.push(data)
                                // 构建消息内容
                                _this.createContent(data.from, null, data.text)
                            }
                        }
                    };
                    //关闭事件
                    socket.onclose = function () {
                        console.log("websocket已关闭");
                    };
                    //发生了错误事件
                    socket.onerror = function () {
                        console.log("websocket发生了错误");
                    }
                }
            }

        }
    }
</script>

<style>
    .tip {
        color: white;
        text-align: center;
        border-radius: 10px;
        font-family: sans-serif;
        padding: 10px;
        width:auto;
        display:inline-block !important;
        display:inline;
    }

    .right {
        background-color: deepskyblue;
    }
    .left {
        background-color: forestgreen;
    }

</style>
