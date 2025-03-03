 
<script setup>
import { showToast, showSuccessToast, showFailToast, showLoadingToast,Overlay ,Popup,Circle} from 'vant';
import { onBeforeMount, onMounted, reactive, ref } from 'vue'
const videoArr = ref([])
const odelSel = ref('')//当前使用的摄像头
const myInterval = ref(null)
const mediaStreamTrack = ref('') // 退出时关闭摄像头
const video_stream = ref('') // 视频stream
const recordedBlobs = ref([]) // 视频音频 blobs
const isRecord = ref(false) // 视频是否正在录制
const content = ref('按住拍摄，点击拍照')
let test = 'c'
const startStauts = ref(true)// 开始录制按钮样式
// video参数
const videoRef = ref(null);
// 画布参数（照片回显）
const cs = ref(null)
const acs = ref(null)
const css = ref(null)
const csss = ref(null)
const cssss = ref(null)
const csssss = ref(null)
const showss=ref(true)
// 回显画布宽高
const canWidth = ref('')
const canHeight = ref('')
const echo_Status = ref(false)
const canvas_echo = ref(null)
// 关闭摄像头
const closeStatus = ref(true)
// 切换按钮状态
const cutStatus = ref(false)
// 照片数量限制
const photoNum = ref([])
// 照片回显数组
const videoList = ref([])
const videoNum = ref(0)
const timeOutEvent = ref(null)
const returns = ref(false)
const imgStatus=ref(false)
const currentRate=ref(0)
const videoStatus=ref(false)
// 录制参数
const isRecording = ref(false)
const videoBlob = ref(null)
const shows=ref(false)
// 前置后置摄像头切换
const cameraStatu = ref(0)
const cameraStatus = ref('user'); 
// 预览内容
let contents = ref({});
const showCenter = ref(false)
onMounted(() => {
  let cedioele = document.getElementsByClassName('camera_video')[0]
  canWidth.value = cedioele.offsetWidth
  canHeight.value = cedioele.offsetHeight
  let canvasList = document.getElementsByClassName('canns')
  cameraStatus.value='user'
  startStream()
})
// 前置后置切换
const changeDevice =async () => {
  cameraStatus.value = cameraStatus.value === 'user' ? 'environment' : 'user';
  closeCamera()
  await startStream();
}
// ------------------------------摄像头开关按钮-------------------------------
// 开启摄像头事件
// const getCamera = () => {
// }
const startStream = async () => {
  console.log(cameraStatu.value);
  const constraints = {
      audio: true,
   video: { facingMode:cameraStatus.value}
  };
 await  navigator.mediaDevices
    .getUserMedia(constraints)
    .then((stream) => {
      // 摄像头开启成功
      startStauts.value = false
      mediaStreamTrack.value = typeof stream.stop === 'function' ? stream : stream.getTracks()[0];
      video_stream.value = stream;
      videoRef.value.srcObject = stream;
    })
    .catch((err) => {
    });
  }
// 关闭摄像头
const closeCamera = () => {
  if (!videoRef.value.srcObject) return;
  let stream = videoRef.value.srcObject;
  let tracks = stream.getTracks();
  tracks.forEach(track => {
    track.stop();
  });
  videoRef.value.srcObject = null;
  startStauts.value = true
  closeStatus.value = true;
}
// ==========================点击拍照============================
const shoot = () => {
  if (startStauts.value == true) {
    showFailToast('请打开摄像头');
  } else {
    if (videoList.value.length < 5) {
      videoList.value.push({
        val: videoRef.value,
        id: videoNum.value++,
        type: 1
      })
      test = test += 's'
      photoNum.value.push(test)
      setTimeout(() => {
        // 大的
          let sc = cs.value[videoList.value.length - 1].getContext('2d');
          const dpr = window.devicePixelRatio || 1;
          const rect = cs.value[videoList.value.length - 1].getBoundingClientRect();  
          cs.value[videoList.value.length - 1].width = rect.width * dpr;
          cs.value[videoList.value.length - 1].height = rect.height * dpr;
          sc.scale(dpr, dpr)
          sc.drawImage(videoList.value[videoList.value.length - 1].val, 0, 0, 400,400);
// 小的
let ssc = acs.value[videoList.value.length - 1].getContext('2d');
          const dprs = window.devicePixelRatio || 1;
          const rects = acs.value[videoList.value.length - 1].getBoundingClientRect();  
          acs.value[videoList.value.length - 1].width = rects.width * dprs;
          acs.value[videoList.value.length - 1].height = rects.height * dprs;
          ssc.scale(dprs, dprs)
          ssc.drawImage(videoList.value[videoList.value.length - 1].val, 0, 0,50,50);
      }, 700)
    } else {
      showFailToast('照片已达上限');
    }
  }
}
// 检测点击按钮事件定时器
let times = null;
const imgUrls=ref('')
const imgUrlss=ref('')
function echo_btn(index) {
  console.log(index);
  shows.value=true
  showss.value=false
  console.log(videoList.value);
  if (videoList.value[index].type == 1) {
  imgStatus.value=true
  videoStatus.value=false
    echo_Status.value = true
    closeStatus.value = true
    cutStatus.value = true
    startStauts.value = false
    const image = cs.value[index].toDataURL("image/png",1.0);
    contents = {
      index: videoNum.value++,
      type: 1,
      url: image,
    }
    imgUrls.value=contents.url
    
    showCenter.value = true
    console.log(contents);
  } else {
videoStatus.value=true
imgStatus.value=false
    echo_Status.value = true
    closeStatus.value = true
    cutStatus.value = true
    startStauts.value = false
    contents = {
      index: videoNum.value++,
      type: 2,
      url: videoList.value[index].val
    }
    console.log(contents);
    imgUrlss.value=contents.url
    showCenter.value = true
  }
}
// 用户是否长按 false 点击、ture 长按
let userStatus = ref(false);
const currentNum = ref(0)
// 手指点击触发
const photosStart = () => {
  console.log(videoList.value.length);
  if (startStauts.value == true) {
    showFailToast('请打开摄像头');
  } else {
    // 判断是否超过5个
    if (videoList.value.length > 5) {
      showFailToast("最多录制5个文件!");
      return;
    }
    times = setTimeout(() => {
      userStatus.value = true;
      pressEvenet();
    }, 500);
  }
 
};
// 长按录像
let mediaRecorder = null;
let inte = null;
let eouts = null;
let stops = false;
const pressEvenet = () => {
  console.log(videoList.value);
  if (videoList.value.length < 5) {
    currentNum.value = 0;
    let chunks = [];
    stops = true;
    mediaRecorder = new MediaRecorder(video_stream.value, {
      mimeType: "video/webm;codecs=vp9",
    });
    mediaRecorder.ondataavailable = (event) => {
      if (event.data && event.data.size > 0) {
        chunks.push(event.data);
      }
    };
    mediaRecorder.onstop = () => {
      const blob = new Blob(chunks, { type: "video/webm" });
      const url = URL.createObjectURL(blob);
      console.log(url);
      videoList.value.push({
        val: url,
        id: videoNum.value++,
        type: 2
      })
      chunks = [];
    };
    mediaRecorder.start();
    inte = setInterval(() => {
      currentNum.value++;
      if (currentNum.value >= 100) {
        showDeleteButton()
      }
    }, 100);
    eouts = setTimeout(() => {
      clearInterval(inte);
      currentNum.value=0
      currentRate.value=0
      showSuccessToast("录制完成");
      mediaRecorder.stop();
      stops = false;
    }, 10000);
  } else {
    showFailToast("最多录制5个文件!");
  }
 
};
const showDeleteButton = () => {
  currentNum.value = 0
  currentRate.value=0
  // 判断是否超过5个
  if (videoList.value.length > 5) {
    showFailToast("最多录制5个文件!");
    return;
  }
  clearTimeout(times);
  pressStop();
}
 
// 停止录制事件
const pressStop = () => {
  if (stops) {
    mediaRecorder.stop();
    showSuccessToast("录制完成");
    stops = false;
    clearInterval(inte);
    clearTimeout(eouts);
  }
};
 
function close(index) {
  console.log(666);
  videoList.value.splice(index, 1)
if(videoList.value.length==1){
  imgStatus.value=false
}
}
function guan(){
  shows.value=false
  showss.value=true
}
 
 
 
 
 
 
 
</script>
 
<template>
  <div style="width:100%;height:100%;background-color:#4c4c4c;z-index:999999" v-show="shows">
    <div style="display:flex;padding-left:95%;background-color:white;cursor:pointer" @click="guan">X</div>
    <van-popup v-model:show="showCenter" style="width:100% ;margin-top:50px">
      <img :src="imgUrls" alt="" v-if="imgStatus" style="width:96%;height:auto;position:absolute;margin-left:auto">
      <video :src="contents.url" v-if="videoStatus" controls autoplay
        :style="{ width: '100%', height: 'auto' }"></video>
    </van-popup>
    </div>
  <div class="camera_box" v-show="showss">
    <!-- 成像区域 -->
    <div class="image_box">
      <!-- <vue-camera ref="camera"></vue-camera> -->
      <video ref="videoRef" autoplay width="100%" height="100%" class="camera_video" style="width:100%;height:100%" ></video>
    </div>
    <!-- 图片预览列表 -->
    <div class="image_list">
      <!-- 加按钮 -->
      <div class="add_btn">
        <img src="../src/assets/adds.png" alt="">
      </div>
      <!-- 列表 -->
      <div class="list">
        <!-- 大的 -->
        <div class="list_item" @click.stop.prevent="echo_btn(index)" v-for="(item, index) in videoList" :key="item.id" style="visibility:hidden;position:fixed">
          <canvas class="canns" ref="cs" width="400" height="400"></canvas>
          <div class="clack" v-if="item.type == 2">
            <video :src="item.val" style="width:100% ;height:100%"></video>
          </div>
          <div class="close_btn">
          </div>
        </div>
<!-- 小的 -->
<div class="list_item" @click.stop.prevent="echo_btn(index)" v-for="(item, index) in videoList" :key="item.id">
  <canvas class="canns" ref="acs" width="50" height="50"></canvas>
  <div class="clack" v-if="item.type == 2">
    <video :src="item.val" style="width:100% ;height:100%"></video>
  </div>
  <div class="close_btn">
  </div>
</div>
 
 
 
 
 
 
 
      </div>
      <!-- 减按钮 -->
      <div class="add_btn" @click.stop.prevent="close(index)">
   <img src="../src/assets/jian.png" alt="">
      </div>
    </div>
<div style="display:flex;align-items:center;justify-content:space-around;margin-top:40px">
  <div> <img src="../src/assets/left.png" alt="" style="width:20px;height:20px"></div>
        <!-- 拍摄操作区域 -->
        <div class="btn_boxs">
          <!-- 图片按钮 -->
          <input type="file" id="file">
          <label for="file" class="image_go">
            <div><van-icon name="photo-o" size="24px" color="#FA9923" /></div>
          </label>
          <!-- 拍摄按钮 -->
          <div class="pach_box" @click.stop.prevent="shoot" @touchstart="photosStart" @touchend="showDeleteButton">
            <van-button plain class="pach_btn">
<div class="addback"></div>
            </van-button>
          </div>
          <!-- 前置后置切换 -->
          <div class="camera_go" ><van-icon name="photograph" size="24px" color="#FA9923" /></div>
          <div class="pach_boxs">
            <van-circle v-model:current-rate="currentRate" :rate="currentNum" :speed="100" :text="text" size="76"
              color="#FA9923" stroke-width="100" />
          </div>
        </div>
  <div> <img src="../src/assets/right.png" @click='changeDevice' alt="" style="width:20px;height:20px"></div>
 
 
 
 
</div>
 
    <div class="message"><b>{{ content }}</b></div>
 
  </div>
</template>
 
<style scoped>
.close_btn {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 10;
  width: 0;
  height: 0;
  border-top: 16px solid #EEEEEE;
  border-left: 16px solid transparent;
  display: flex;
  justify-content: center;
  align-items: center;
}
 
.close_btn>img {
  width: 8px;
  height: 8px;
  position: absolute;
  top: -13px;
  right: 1px;
}
 
.clack {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}
 
.clack>img {
  width: 100%;
  height: 100%;
}
 
.canvas_echo {
  position: absolute;
  top: 0;
  left: 0;
}
 
.camera_box {
 
}
 
.success_btn {
  padding: 10px;
}
 
.start_live {
  padding: 10px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 5;
}
 
.end_live {
  padding: 10px;
  position: absolute;
  bottom: 0;
  right: 0;
  transform: translate(-50%, -50%);
  z-index: 5;
}
 
.cut_live {
  padding: 10px;
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 5;
}
 
.image_box {
  width: 100%;
  height: 60vh;
  position: relative;
  border-radius: 0 0 10px 10px;
  border: 1px solid #EEEEEE;
}
 
.live_window {
  width: 100%;
 
}
 
.back_btn {
  width: 20px;
  height: 20px;
  border: 50%;
  opacity: 0.5;
}
 
.btn_box {
  width: 100%;
  display: flex;
  justify-content: space-between;
  position: absolute;
  top: 0;
  font-size: 1rem;
}
 
.image_list {
  width: 100%;
  height: 50px;
  margin-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}
 
.btn_boxs {
 
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}
 
.image_go {
  width: 20%;
}
 
.camera_go {
  width: 20%;
  display: flex;
  justify-content: right;
}
 
 
 
.pach_boxs {
  width: 100%;
  height: 100%;
  position: absolute;
  border-radius: 50%;
  z-index: -1;
  display: flex;
  justify-content: center;
  align-items: center;
}
 
.pach_icon {
  width: 80%;
  height: 80%;
}
 
.pach_btn {
  width: 70px;
  height: 65px;
  border-radius: 50%;
  background-color: #e6e6e6;
  display: flex;
  justify-content: center;
  margin-top: -5px;
  align-items: center;
}
 
.pach_img {
  width: 40px;
  height: 40px;
}
.addback{
  width: 20px;
  height: 20px;
  background-color: rgba(205, 205, 205, 1);
  border-radius: 50%;
}
 
.add_btn {
  width: 30px;
  height: 30px;
  background-color: #FA9A24;
  display: flex;
  font-size: 20px;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
  margin-left: 10px;
}
 
.add_btn>img {
  width: 12px;
  height: 12px;
}
 
.list {
  display: flex;
  align-items: center;
}
 
.list_item {
  width: 50px;
  height: 50px;
  background-color: #EEEEEE;
  margin-left: 10px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
}
 
#file {
  display: none;
}
 
.message {
  margin-top: 30px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
 
 
 
 
 