<template>
  <div class="user-list">
    <div class="title">
      <p>Member List</p>
    </div>
    <!-- 멤버리스트-->
    <ul class="list-video" v-show="showModel == 0">
      <!-- 자신의 비디오-->
      <li class="li-my">
        <div class="avatar avatar-name">
          {{ $store.state.data.userName }}
        </div>
        <video autoplay class="mirrorMode myUserId" :id="$store.state.data.userId" @click="switchScreen(myUserId)"></video>
        <p class="name">{{ $store.state.data.userName }} </p>
      </li>
      <!-- 자신이외의 비디오-->
      <li v-for="v in $store.state.data.userList" :key="v.userId">
        <!-- [:userInfo="v"]: props로 유저정보를 hvideo로 넘긴다 -->
        <!-- [@switchScreen="switchScreen"]: hvideo에서 userId를 받는다 -->
        <hvideo @switchScreen="switchScreen" :userInfo="v"></hvideo>
      </li>
    </ul>
    <ul class="list-txt" v-show="showModel == 1">
        <li>
          <p class="onlyname">{{ $store.state.data.userName }}</p>
        </li>
      <li v-for="v in $store.state.data.userList" :key="v.userId">
        <p class="onlyname">{{ v.displayName }}</p>
      </li>
    </ul>
  </div>
</template>

<script>
import RTCClient from "../core/rtc-client";
import Util from "../core/utils/utils";
export default {
  data() {
    return {
      showModel: 0,
      myUserId: null,
      count: 0,
    };
  },
  mounted() {
    this.$nextTick(() => {});
  },
  methods: {
    switchScreen(userId) {
      let subUserId = document.getElementById("localVideo").getAttribute("subUserId");
      this.myUserId = RTCClient.instance.userId;

      this.$nextTick(() => {
        if (subUserId) {
          var _userId = subUserId;
          RTCClient.instance.subscribe(_userId).then((code) => {
            RTCClient.instance.setDisplayRemoteVideo(_userId,document.getElementById(_userId), code);
          });
        }
        if (userId == this.myUserId || userId == subUserId) {
          if (userId == this.myUserId) {
            setTimeout(() => {
              if (this.$store.state.data.isPublishScreen) {
                document.getElementById("localVideo").srcObject = RTCClient.instance.screenStream;
              } else {
                document.getElementById("localVideo").srcObject = AppConfig.localStream;
              }
            }, 200);
            document.getElementById("localVideo").removeAttribute("subUserId");
            hvuex({
              isSwitchScreen: false,
            });
          }
          return;
        } else {
          document.getElementById("localVideo").setAttribute("subUserId", userId);
          RTCClient.instance.subscribeLarge(userId).then((code) => {
            setTimeout(() => {
              // 원격 비디오에 대한 렌더링 창 및 그리기 매개변수 설정
              RTCClient.instance.setDisplayRemoteVideo(
                userId,
                document.getElementById("localVideo"),
                code
              );
            }, 200);
          }).catch((error) => {
              console.log('error: ', error);
          });
          hvuex({
            isSwitchScreen: true,
          });
        }
      });
    },
  },
  watch: {
    "$store.state.data.switchUserId"(newVal, oldVal) {
      if (newVal) {
        this.switchScreen(newVal);  
        hvuex({ switchUserId: null });
      }
    },
    "$store.state.data.userList"(newVal, oldVal) {
      if (JSON.stringify(newVal) == JSON.stringify(oldVal)) {
        return;
      }

      // 접속한 2사람 전부 메인화면이 안나옴
      // this.switchScreen(newVal[0].userId); 

      // 나중에 접속한 사람의 메인화면만 변경
      // this.switchScreen(newVal[0].userId);
      // hvuex({ switchUserId: null });

      // 접속한 2사람 전부 자신의 화면이 메인화면에 나옴
      // this.switchScreen(newVal);
      // hvuex({ switchUserId: null });

      // 접속한 2사람 전부 메인화면이 안나옴
      // this.switchScreen(newVal[0].userId);
      // hvuex({ switchUserId: null });
      // hvideo.vue 파일도 변경
      // this.$emit("switchScreen", this.userInfo.userId);

      // 처음에 접속한 사람의 메인화면만 변경 
      // hvideo.vue 파일도 변경

      
      // 상대방 이름 취득(EC시스템의 Order관리의 고객을 담당한 스텝이름)
      hvuex({
        guestName: newVal[0].displayName 
      });

      
      let subUserId = document.getElementById("localVideo").getAttribute("subUserId");
      if (subUserId) {
        if (!RTCClient.instance.getUserInfo(subUserId)) {
          if (this.$store.state.data.isPublishScreen) {
            document.getElementById("localVideo").srcObject = RTCClient.instance.screenStream;
            if (this.$store.state.data.isSwitchScreen) {
              document.getElementById(RTCClient.instance.userId).srcObject = AppConfig.localStream;
            }
          } else {
            document.getElementById("localVideo").srcObject = AppConfig.localStream;
          }
          document.getElementById("localVideo").removeAttribute("subUserId");
          hvuex({
            isSwitchScreen: false,
          });
        }
      }
    },
  },
};
</script>

<style lang="scss">
.user-list {
  //width: vw(218);
  width: 196px;
  height: vh(720-40-104); //position: fixed;
  top: 40px;
  right: 0;
  bottom: 104px;
  padding: 0 13px;
  box-sizing: border-box;
  .title {
    margin: 12px 0 12px 0;
    font-size: 16px;
    display: flex;
    text-align: center;
    cursor: pointer;
    p {
      position: relative;
      flex: 1;
      padding: 10px 0;
    }
    p::after {
      display: block;
      content: "";
      height: 1px;
      background-color: black;
      margin-top: 20px;
    }
    .on::after {
      height: 4px;
      background-color: #013ebe;
      margin-top: 18px;
    }
  }
  .list-video {
    overflow-x: scroll;
    height: vh(720-40-104-80);
    padding-bottom: vh(30);
    .li-my {
      position: relative;
      $height: 126px; // 31.1vh;
      height: $height;
      overflow: hidden;
      width: vh(128/3 * 4);
      margin-bottom: 10px;
      .avatar {
        position: absolute;
        text-align: center;
        top: 0;
        width: 100%;
        line-height: $height;
        bottom: 0;
        color: #333333;
        background-position: center;
        background-size: contain;
        background-repeat: no-repeat;
        background-color: #409eff;
        // background-color: rgba(89, 89, 89, $alpha: 1);
        img {
          display: none;
        }
      }
      .avatar-name {
        color: white;
        font-size: vh(40);
        text-align: center;
        line-height: vh(110);
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
      video {
        position: absolute;
        top: 0;
        width: 100%;
        height: $height;
        object-fit: fill;
        background-color: transparent;
      }
      .name {
        position: absolute;
        bottom: 10px;
        left: 10px;
        font-size: 18px;
        color: white;
      }
    }
  }
  .list-txt {
    p {
      line-height: 48px;
      cursor: pointer;
      text-indent: 30px;
    }
    p:hover {
      background-color: #d2e1ff;
    }
  }
}
</style>