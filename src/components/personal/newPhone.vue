<template>
    <div class="newPhone">
        <div class="borderccc bR5 btnWidth plr15 ptb10">
            <input class="flex" v-model="phoneNum" type="text" placeholder="请输入新的手机号"/>
        </div>
        <div class="borderccc bR5 btnWidth">
            <van-cell-group class="ml5">
                <van-field
                    center
                    v-model="sms"
                    placeholder="请输入验证码"
                    icon="clear"
                    @click-icon="sms = ''">
                    <van-button @click="getValidateCode" v-if="is_show"  slot="button" size="normal" type="primary" class="bgGreen">{{getCodetext}}</van-button>
                    <van-button v-else slot="button" size="normal" class="bg999">已发送{{last_time}}秒</van-button>
                </van-field>
            </van-cell-group>
        </div>
        <button class="bgGreen btnCommit" @click="commit">更换</button>
        <ymDialog ref="dialog" :modal="modal" />
    </div>
</template>
<script>
import ymDialog from "../common/dialog";
import {Toast} from 'vant'
export default {
  data() {
    return {
      phoneNum:'',
      sms:'',
      auth:'',
      //倒计时
      is_show: true,
      last_time: "",
      getCodetext: "获取验证码",
      modal: {
        confirmText: "立即登录",
        contentText: "更换成功",
        red: false,
        showCancel: false
      }
    };
  },
  mounted() {
    this.auth = this.$route.params.auth;
    console.log(this.auth)
  },
  methods: {
    getValidateCode() {
      let _this = this;
        _this
        .$fetch(_this.GLOBAL.base_url + "sms", {
          mobile: _this.phoneNum,
          type: "mobile"
        })
        .then(res => {
          console.log(res,'手机号');
          if (res.code == 200) {
            _this.is_show = !_this.is_show;
            _this.GLOBAL.countdown(_this);
          }else{
            Toast(res.msg)
          }
        })
        .catch(err => {
          console.log(err);
          Toast('网络请求超时')
        });
      
    },
    commit(){
        let _this = this;
        if(_this.phoneNum == ''){
          Toast('手机号不能为空')
        }else if (this.sms == "") {
          Toast("请填写验证码");
        }  else {
          _this
            .$post(_this.GLOBAL.base_url + "mobile",{
              token:_this.GLOBAL.token,
              mobile: _this.phoneNum,
              code: Number(_this.sms),
              auth:_this.auth
            })
            .then(res => {
              console.log(res,'修改手机号');
              if (res.code == 200) {
                _this.$refs.dialog.confirm().then(()=>{
                  _this.$router.replace(
                    {name:'login',params:{title:'登录'}}
                  )
                  sessionStorage.clear();
                }).catch(()=>{
                })
              }else{
                Toast(res.msg);
              }
            })
            .catch(err => {
              console.log(err);
            });
        }
    }
  },
  components: {
    ymDialog
  }
};
</script>
<style lang="less" scoped>
.newPhone {
  margin-top: 100px;
  .btnWidth {
    width: 290px;
    height: 44px;
    margin: auto;
    ::-webkit-input-placeholder {
      color: #ccc;
    }
  }
  .van-cell {
    padding: 0 !important;
  }
  
  .van-button--primary {
    border: 0;
  }
  .colorccc{
  color: #ccc;
}
  .borderccc {
    border: 1px solid #ccc;
  }
  .bg999 {
    background-color: #999;
  }
  .btnWidth + .btnWidth {
    margin-top: 25px;
  }
  .btnCommit {
    width: 290px;
    height: 44px;
    margin: auto;
    border-radius: 5px;
    margin-top: 80px;
    color: #fff;
    font-size: 20px;
  }
}
</style>


