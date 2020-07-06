<template>
<div class="chatroom">
    <div class="header">
        <div class="header__name">
            匿名聊天
            <input type="text" v-model="name" style="margin-left:10px;" placeholder="請輸入您的暱稱" />
        </div>
    </div>
    <div id="chatbox" class="chatbox">
        <div class="chat" v-for="(data, key) in datas" :key="key">
            <div class="flex--right" v-if="data.uid === name">
                <div v-if="data.type === 'text' " class="chat__message chat--self">
                    {{data.message}}
                </div>
                <div v-else-if="data.type === 'image'" class="chat__image--seif" style="margin-right:10px;">
                    <img :src="data.message" width="100px" height="100px" >
                </div>
            </div>
            <div class="flex--left" v-else>
                <img style="display:block;" class="chat--other chat__profile" src="../assets/641.jpg" width="40" height="40">
                <div class="chat--other" style="width:100%;" >
                    <div class="chat__name">
                        {{data.uid}}
                    </div>
                    <div class="flex--left">
                        <div v-if="data.type === 'text' " class="chat__message" style="background-color: #E3E8EB;">
                            {{data.message}}
                        </div>
                        <div v-else-if="data.type === 'image'" class="chat__image--other">
                            <img :src="data.message" width="100px" height="100px" >
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="option"> 

    </div>
    <div class="input">
        <label>
            <input id="imageUpload" type="file" accept="image/*" style="display:none;" @change="uploadImage($event)">
            <img src="../assets/picture.png" height="40px" width="40px;" >
        </label>
        <textarea class="input__message " v-model="message" rows="1" @keypress.enter="sendMessage($event)"> </textarea>
        <img src="../assets/play-button.png" height="40px" width="40px;" 
            class="input__submit" @click="sendMessage($event)">
    </div>
</div>
</template>
<script>
let msgRef = firebase.database().ref('/message/');
let storageRef = firebase.storage().ref('/images/');
export default {
    data() {
        return {
            name:'',
            message:'',
            datas:[],
            loading: false,
        }
    },
    methods: {
        sendMessage(e) {
            if (!this.name) {
                alert('請輸入姓名');
            }
            else if (!this.message) {
                alert('請輸入訊息');
            }
            else {
                const msg = {
                    uid: this.name,
                    message: this.message,
                    type:'text',
                    unitTime:'',
                }
                msgRef.push(msg);
                this.message = '';
                e.preventDefault();
            }
        },
        scrollDown() {
            const roomBody = document.querySelector('#chatbox');
            roomBody.scrollTop = roomBody.scrollHeight;
        },
        autogrow(textarea) {
            let adjustedHeight = textarea.clientHeight;
            adjustedHeight = Math.max(textarea.scrollHeight,adjustedHeight);
            if (adjustedHeight > textarea.clientHeight){
                textarea.style.height= adjustedHeight + 'px';
            }
        },
        uploadImage(e) {
            const vm = this;
            if (vm.name) {
                const file = e.target.files[0];
                e.target.value=''; //use to upload same file
                const fileName = Math.floor(Date.now() / 1000) + `_${file.name}`;
                const metadata = {
                    contentType:'image/*'
                };
                const uploadTask = storageRef.child(fileName).put(file, metadata);
                uploadTask.on(firebase.storage.TaskEvent.STATE_CHANGED, function(snapshot) {
                    let progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
                    console.log('Upload is ' + progress + '% done');
                }, function(error) {
                }, function () {
                    uploadTask.snapshot.ref.getDownloadURL().then(function(downloadURL) {
                        const msg = {
                            uid: vm.name,
                            message: downloadURL,
                            type: 'image',
                        }
                        msgRef.push(msg);
                    })
                })
            }
            else {
                alert('請輸入姓名');
                e.target.value='';
                e.preventDefault();
            }
        },
    },
    mounted() {
        msgRef.on('value', res => {
            const val = res.val();
            const data = Object.keys(val).map(key => ({...val[key]}));
            this.datas = data;
        })
    },
    updated() {
        this.scrollDown();
    }    
}
</script>
<style scoped>
* {
  font-family: '微軟正黑體';
  margin: auto;
}
.chatroom {
    width:500px;
    margin:auto;
    border: 1px solid;
}
.header {
    width:100%;
    height:50px;
    background-color: #2B364B;
    color:white;
}
.header__name {
    float:left;
    font-size:20px;
    margin-top:5px;
    margin-left: 5px;;
}
.chatbox {
    width:100%;
    height:500px;
    overflow: scroll;
    overflow-x:hidden;
}
.option {
    height:35px;
    background-color:#E7E7E7 ;
}
textarea:focus {
    outline: none;
}
.input {
    display:flex;
    justify-content: flex-end;
    margin:3px;
}
.input__message {
    border-radius: 30px;
    font-size:14px;
    padding:10px;
    width:70%;
    margin-right: 0px;
    margin-left: 30px;
}
.input__submit {
    height:40px;
    margin-right:10px;
    margin-left:10px;
}
.chat {
    margin:10px;
}
.chat__message {
    text-align: left;
    max-width:60%;
    padding:7px;
    border-radius:15px;
    word-break: break-all;
    margin:0px;
}
.chat__image--other {
    margin-top:10px;
    margin-left:10px;
}
.chat__image--seif {
    margin-top:10px;
    margin-left:10px;
}
.chat--self {
    margin-right:10px;
    background-color: #AFF47E;
}
.flex--right {
    display:flex;
    justify-content: flex-end;
}
.flex--left {
    display:flex;
    justify-content: flex-start;
}
.chat--other {
    margin-left:10px;
    margin-right: 0px;
    margin-top: 0px;
    margin-bottom: 0px;
}
.chat__profile {
    border-radius: 100%;
}
.chat__name {
    text-align: left;
    font-size:10px;
    background-color: white;
}
</style>