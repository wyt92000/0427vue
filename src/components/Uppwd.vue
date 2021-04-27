<template>
    <div class="box">
        <img src="../../static/image/13.jpg" alt="">
        <div class="register">
            <div class="register_box">
                <div class="register-title">忘记密码</div>
                <div class="inp">
                    <input v-model="phone"  type="text" placeholder="手机号码" class="user" @blur="check_phone">
                    <input v-model="password" type="password" placeholder="请输入登录密码" class="user" @blur="check_password">
                    <input v-model="re_pwd" type="password" placeholder="请再次输入密码" class="user" @blur="check_re_pwd">
                    <div id="geetest"></div>
                    <div class="sms-box">
                        <input v-model="code" type="text" maxlength="6" placeholder="请输入验证码" class="user">
                        <div class="sms-btn" @click="time_down" :disabled="disabled">{{ btnText }}</div>
                    </div>
                    <button class="register_btn" @click="forget_pwd">提交</button>
                    <p class="go_login">已有账号
                        <!--                        <router-link to="/login">直接登录</router-link>-->
                        <span>直接登录</span>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "Uppwd",
     data() {
        return {
            phone: "",
            password: "",
            re_pwd:"",

            code: "",
            disabled: false,
            interval:undefined,
            totalCount:0,

            flag: true,
        }
    },
    methods:{
        // 验证手机号格式
        check_phone() {
             if(this.phone === ""){
                 this.$alert("手机号不能为空，请确认", "警告");
                 this.flag = false;
                 return this.flag;
            }
            else if(!/^1[34578]\d{9}$/.test(this.phone)){
                this.$alert("手机号格式有误，请确认", "警告");
                this.flag = false;
                return this.flag;
            }else {
                // 向后端发请求判断当前输入手机号是否存在
                this.$axios({
                    url: this.$settings.HOST+"user/check_phone_up/",
                    method: 'get',
                    params: {
                        phone: this.phone
                    }
                }).then(res=>{
                    console.log(res.data);
                }).catch(error=>{
                    this.flag = false
                    console.log(error.response);
                    this.$message.error("您的信息有误");
                })
            }
            return this.flag;
        },

        // 验证密码格式
        check_password(){
            if (this.password === ""){
                this.$alert("密码不能为空，请确认", "警告");
                this.flag = false;
                return this.flag;
            }
            else if(!(/^[a-zA-Z][a-zA-Z0-9]{6,16}$/.test(this.password))){
                this.$alert("密码格式有误，请确认", "警告");
                this.flag = false;
                return this.flag;
            }
            return this.flag;
        },

        // 验证重复密码
        check_re_pwd() {
             if(this.re_pwd === ""){
                 this.$alert("重复密码不能为空，请确认", "警告");
                 this.flag = false;
                 return this.flag;
            }
            else if(this.re_pwd !== this.password){
                this.$alert("两次密码不同，请确认", "警告");
                this.flag = false;
                return this.flag;
            }
            return this.flag;
        },

        // 按钮60秒倒计时
        time_down(){
             // 格式正确即可发送信息
            console.log(this.check_phone());
            console.log(this.check_re_pwd());
            console.log(this.check_password());
            if (this.check_phone() === this.check_password() === this.check_re_pwd()) {
                this.disabled=true
                this.totalCount=60
                this.get_code()    //60秒过倒计时过后才能调用的事件
                this.interval=setInterval(()=>{
                    this.totalCount--
                    if(this.totalCount === 0){
                    clearInterval(this.interval)
                    this.disabled=false
                  }
                },1000);
             }else {
                 this.$alert("请检查输入信息", "警告");
            }
        },

        // 获取验证码
        get_code() {
            this.$axios({
                url: this.$settings.HOST + "user/message/",
                method: 'get',
                params: {
                    phone: this.phone
                }
            }).then(res => {
                console.log(res.data);
            }).catch(error => {
                console.log(error.response);
                this.$message.error("短信发送失败或已经发送过短信");
            })
        },

        // 发送更新密码请求
        forget_pwd(){
            this.$axios({
                url: this.$settings.HOST + "user/forget_pwd/" + "?phone=" +this.phone,
                method: 'patch',
                data: {
                    phone: this.phone,
                    password: this.password,
                    sms_code: this.code,
                }
            }).then(res => {
                console.log(res);
                // 页面跳转
                let self = this;
                this.$alert("密码修改成功", "百知教育", {
                    callback() {
                        self.$router.push("/login");
                    }
                })

            }).catch(error => {
                console.log(error);
            })
        },


        },
    computed: {
        // 计算属性监视
        btnText(){
            return this.totalCount !==0? `${this.totalCount}秒再次获取`: "获取验证码"
            }
        },

}
</script>

<style scoped>
.box {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: hidden;
}

.box img {
    width: 100%;
    min-height: 100%;
}

.box .register {
    position: absolute;
    width: 500px;
    height: 400px;
    top: 0;
    left: 0;
    margin: auto;
    right: 0;
    bottom: 0;
    top: -338px;
}

.register .register-title {
    width: 100%;
    font-size: 24px;
    text-align: center;
    padding-top: 30px;
    padding-bottom: 30px;
    color: #4a4a4a;
    letter-spacing: .39px;
}

.register-title img {
    width: 190px;
    height: auto;
}

.register-title p {
    font-family: PingFangSC-Regular;
    font-size: 18px;
    color: #fff;
    letter-spacing: .29px;
    padding-top: 10px;
    padding-bottom: 50px;
}

.register_box {
    width: 400px;
    height: auto;
    background: #fff;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, .5);
    border-radius: 4px;
    margin: 0 auto;
    padding-bottom: 40px;
}

.register_box .title {
    font-size: 20px;
    color: #9b9b9b;
    letter-spacing: .32px;
    border-bottom: 1px solid #e6e6e6;
    display: flex;
    justify-content: space-around;
    padding: 50px 60px 0 60px;
    margin-bottom: 20px;
    cursor: pointer;
}

.register_box .title span:nth-of-type(1) {
    color: #4a4a4a;
    border-bottom: 2px solid #84cc39;
}

.inp {
    width: 350px;
    margin: 0 auto;
}

.inp input {
    border: 0;
    outline: 0;
    width: 100%;
    height: 45px;
    border-radius: 4px;
    border: 1px solid #d9d9d9;
    text-indent: 20px;
    font-size: 14px;
    background: #fff !important;
}

.inp input.user {
    margin-bottom: 16px;
}

.inp .rember {
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
    margin-top: 10px;
}

.inp .rember p:first-of-type {
    font-size: 12px;
    color: #4a4a4a;
    letter-spacing: .19px;
    margin-left: 22px;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-align: center;
    align-items: center;
    /*position: relative;*/
}

.inp .rember p:nth-of-type(2) {
    font-size: 14px;
    color: #9b9b9b;
    letter-spacing: .19px;
    cursor: pointer;
}

.inp .rember input {
    outline: 0;
    width: 30px;
    height: 45px;
    border-radius: 4px;
    border: 1px solid #d9d9d9;
    text-indent: 20px;
    font-size: 14px;
    background: #fff !important;
}

.inp .rember p span {
    display: inline-block;
    font-size: 12px;
    width: 100px;
    /*position: absolute;*/
    /*left: 20px;*/

}

#geetest {
    margin-top: 20px;
}

.register_btn {
    width: 100%;
    height: 45px;
    background: #84cc39;
    border-radius: 5px;
    font-size: 16px;
    color: #fff;
    letter-spacing: .26px;
    margin-top: 30px;
}

.inp .go_login {
    text-align: center;
    font-size: 14px;
    color: #9b9b9b;
    letter-spacing: .26px;
    padding-top: 20px;
}

.inp .go_login span {
    color: #84cc39;
    cursor: pointer;
}

.sms-box {
    position: relative;
}

.sms-btn {
    font-size: 14px;
    color: #ffc210;
    letter-spacing: .26px;
    position: absolute;
    right: 16px;
    top: 10px;
    cursor: pointer;
    overflow: hidden;
    background: #fff;
    border-left: 1px solid #484848;
    padding-left: 16px;
    padding-bottom: 4px;
}
</style>
