<template>
  <div class="login" :style="'background-image:url(' + bgUrl + ')'">
    <div class="main">
      <div class="main-title">
        <i class="iconfont icon-rtcyinshipintongxin"></i>
        <span>{{$t("login.title")}}</span>
      </div>
      <div class="main-input shadow">
        <input type="text" placeholder="请输入用户名" v-model="displayName"  readonly/>
        <input type="text" autocomplete="off" placeholder="room number" v-model="room" id="channel" v-on:keyup.enter="submit()" />
        <button class="hui hui-btn" :disabled="room.length == 0 && displayName" @click="submit()">{{$t("login.join")}}</button>
      </div>
      <div class="main-button">
        <button type="button" :disabled="room.length > 0 || displayName.length === 0" @click=" room = Math.random().toFixed(5).slice(-5)">
          {{$t("login.createRoom")}}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import bgUrl from "../../assets/icon/bg.jpg";
import RTCClient from "../../core/rtc-client";

export default {
  data() {
    return {
      room: "",
      displayName: "", // 이름
      bgUrl: bgUrl,
    };
  },
  created() {
    // EC에서 접속할때 전송한 사용자 ID의 유무 판단
    if (!this.$route.query.userId) {
      this.$router.push('/err');
      return;
    }
  },
  mounted() {
    // 다국어 설정
    if(this.$route.query.i18nextLng) {
      this.$i18n.locale = this.$route.query.i18nextLng;
    }

    // 사용자명(스템 또는 관리자) 설정
    this.displayName = this.$route.query.name;
  },
  methods: {
    // 채널가입
    submit() {
      var reg = new RegExp(/^([0-9]{1,12})?$/g);
      // 비디오 채팅에서 이름과 방번호를 입력해서 룸에 접속하는 경우
      if (!reg.test(this.room)) {
        this.$message("会议码格式不正确, 请输入12位以内纯数字");
        return;
      }

      hvuex({ classNum: this.room, userName: this.displayName, loginUserId: this.$route.query.userId, type: this.$route.query.type });
      
      // 페이지 이동
      this.$router.push("/meet");
    },
  },
};
</script>

<style lang="scss">
.login {
  min-height: 100vh;
  position: relative;
  background-size: 100% 100%;
  //background-color: rgb(16, 56, 207);
  .publisher-btn {
    height: 26px;
    line-height: 26px;
    background-color: #006eff;
    border: 0;
    cursor: pointer;
    color: white;
    border-radius: 5px;
    margin-left: 200px;
  }
  .main {
    width: 500px;
    background-color: #fff;
    padding: 30px 40px; //  58px 110px;
    border-radius: 8px;
    box-sizing: border-box;
    position: absolute;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    font-size: 14px;
    .disabled {
      border: 1px solid #ddd !important;
      background-color: #f5f5f5 !important;
      color: #aca899 !important;
    }
    input[type="checkbox"] {
      margin-right: 8px;
    }
    .main-title {
      font-size: 32px;
      text-align: center;
      color: #000000;
      letter-spacing: 1px;
      .iconfont {
        font-size: 64px;
        vertical-align: middle;
        margin-right: 10px;
      }
    }
    .main-input {
      margin: 38px 0 24px;
      position: relative;
      input,
      select {
        padding: 0 14px;
        width: 100%;
        box-sizing: border-box;
        line-height: 38.5px;
        height: 56px;
        color: #111111;
        border: solid 1px #ddd;
        position: relative;
        margin-bottom: 16px;
        touch-action: none;
        border-radius: 3.5px;
        font-size: 21px;
        caret-color: #3296fa;
      }
      input:focus {
        outline: none;
        border: 1px solid #3296fa;
      }
      input::-webkit-input-placeholder {
        color: #8a8a8a;
      }
      .hui-btn {
        position: absolute;
        right: 0px;
        top: 72px;
        font-family: PingFangSC-Regular;
        font-size: 22px;
        padding: 0 14px;
        width: 120px;
        box-sizing: border-box;
        line-height: 56px;
        height: 56px;
        background-color: #3296fa;
        border: solid 1px #3296fa;
        outline: none;
        color: white;
        letter-spacing: 1px;
        border-radius: 0 3.5px 3.5px 0;
        cursor: pointer;
      }
      .hui-btn:disabled {
        opacity: 0.5;
      }
    }
    .live-input {
      display: inline-block;
      position: relative;
      bottom: 15px;
      label {
        font-size: 15px;
        color: #888;
      }
      input::before {
        border-color: #888;
      }
    }
    .main-button {
      button {
        outline: none;
        padding: 0 14px;
        width: 100%;
        box-sizing: border-box;
        line-height: 56px;
        height: 56px;
        background-color: #3296fa;
        border: solid 1px #3296fa;
        color: white;
        letter-spacing: 1px;
        font-size: 22px;
        border-radius: 3.5px;
        cursor: pointer;
      }
      button:disabled {
        opacity: 0.5;
      }
    }
  }
}
</style>
