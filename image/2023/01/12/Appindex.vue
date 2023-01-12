<template>
  <el-container>
    <el-header class="head-bar">
      <span><img class="iot-icon" src="/iot.svg" /></span>
      <span class="login_title">&nbsp;智能家居管理系统</span>
    </el-header>
    <el-container>
      <el-aside class="side-bar">
        <el-scrollbar class="side-bar-scroll">
          <div style="padding: 0px 30px 0px 10px;">
            <el-menu active-text-color="#04ccaf" background-color="#252525" text-color="#fff" default-active="" @select="handleSelect">
              <el-menu-item index="1">
                <template #title>
                  设备列表
                </template>
              </el-menu-item>
              <el-sub-menu index="2">
                <template #title>
                  设备管理面板
                </template>
                <el-row>
                  <el-col :span="16"></el-col>
                  <el-col :span="4">
                    <el-button class="circle-button" @click="onDeleteSiteClick"><img src="/delete.png"
                        style="width: 15px; height: 15px;"></el-button>
                  </el-col>
                  <!-- <ConfirmDialog>删除后不可恢复，你确定要删除吗？</ConfirmDialog> -->
                  <el-col :span="4">
                    <el-button class="circle-button" @click="onAddSiteClick">+</el-button>
                  </el-col>
                </el-row>
                <span v-for="(site,index) in sites" class="site-option" :id="siteIndex(index)">
                  <el-menu-item :index="siteIndex(index)">
                    <template #title>
                      {{ site.name }}
                    </template>
                  </el-menu-item>
                </span>
              </el-sub-menu>
            </el-menu>
          </div>
        </el-scrollbar>
      </el-aside>
      <el-main style="margin: 0px 0px 0px 30px; ">
        <el-scrollbar class="main-bar-scroll" :style="{ width: mainWidth + 'px' }">
          <el-collapse>
            <!-- <el-collapse-item title="房间1">
              <canvas>
                房间占位符
              </canvas>
              <el-upload v-model:file-list="siteImageFile1" action="/api/room/image" :limit="1" :show-file-list="false">
                <el-button>上传户型图</el-button>

              </el-upload>
            </el-collapse-item>
            <el-collapse-item title="房间1">
              <el-card>
                房间占位符
                <el-button @click="dialogVisible = true">打开对话框</el-button>
                <el-dialog v-model="dialogVisible">
                  新建房间
                  <el-form label-width="auto">
                    <el-form-item label="房间名">
                      <el-input></el-input>
                    </el-form-item>
                    <el-form-item label="长">
                      <el-input></el-input>
                    </el-form-item>
                    <el-form-item label="宽">
                      <el-input></el-input>
                    </el-form-item>
                    <el-upload v-model:file-list="siteImageFile2" action="/api/room/image" :limit="1"
                      :show-file-list="false">
                      <el-button>上传户型图</el-button>

                    </el-upload>
                    <el-form-item>
                      <el-button class="bold nonprimary"
                        style="width:40%; background: #252525;border: 1mm solid #007f6e; border-radius: 1ch;"
                        v-on:click="dialogVisible = false">取消</el-button>
                      <el-col :span="4"></el-col>
                      <el-button class="bold" type="primary" style="width:40%; background: #007f6e;border: none"
                        v-on:click="dialogVisible = false">确定</el-button>
                    </el-form-item>

                  </el-form>
                </el-dialog>
              </el-card>
            </el-collapse-item> -->
            <div v-if="(currentSite == null)"></div>
            <div v-else-if="checkHasRoom">
              <span v-for="(room, index) in rooms" :id="roomIndex(index)">
                <el-collapse-item>
                  <template #title>
                    <el-row style="width:100%">
                      <el-col :span="4">{{room.name}}</el-col>
                      <el-col :span="2" :offset="18"><el-button type="danger" 
                        @click.stop.prevent="(deleteRoomDialogVisible = true, deleteRoomIndex = index)">
                        <img src="/delete.png" style="width: 15px; height: 15px;"/></el-button></el-col>
                    </el-row>
                    
                  </template>
                  <el-card>
                    <el-form>
                      <el-row>
                        <el-col :span="2" style="font-size:large">
                          设备列表
                        </el-col>
                        <el-col :offset="21" :span="1">
                          <el-button type="success" @click="onAddDeviceClick(index, 0, 0)">+</el-button>
                        </el-col>
                      </el-row>
                      <el-form-item></el-form-item>
                      <el-row>
                        <el-col :span="4">设备名称</el-col>
                        <el-col :span="4">设备类型</el-col>
                        <el-col :span="4">设备状态</el-col>
                      </el-row>
                      <el-row v-for="(device, deviceIndex) in room.devices" style="height: 35px">
                        <el-col :span="4"> {{ device.name }} </el-col>
                        <el-col :span="4"> {{ typeOptions[device.type].label }}</el-col>
                        <el-col :span="4">{{ deviceValueMap(device)}}</el-col>
                        <el-col :offset="11" :span="1">
                          <el-button type="danger" @click="onDeleteDeviceClick(index, deviceIndex)"><img src="/delete.png" style="width: 15px; height: 15px;"></el-button>
                        </el-col>
                      </el-row>
                    </el-form>
                  </el-card>
                </el-collapse-item>
              </span>
              <el-button @click="(addRoomDialogVisible = true)">新建房间</el-button>
            </div>
            <div v-else>
              <div style="color:#fff">场所图</div>
              <canvas :id="currentSiteCanvas" style="background-color:#fff"></canvas>
              <el-checkbox-button v-model="isAddDevice">添加新设备</el-checkbox-button>
              <el-form :inline="true">
                <el-form-item label="设备名">
                  <el-input v-model="deleteDeviceName"></el-input>
                </el-form-item>
                <el-form-item>
                  <el-select v-model="deleteDeviceType" placeholder="请选择">
                    <el-option v-for="item in typeOptions" :label="item.label" :value="item.value"/>
                  </el-select>
                </el-form-item>
                <el-form-item><el-button type="danger" @click="onDeleteSiteDeviceClick">删除设备</el-button></el-form-item>
              </el-form>
            </div>
          </el-collapse>
        </el-scrollbar>
      </el-main>
    </el-container>
  </el-container>


  <el-dialog v-model="deleteConfirmDialogVisible">
    <el-form>
      <el-form-item>
        删除后不可恢复，你确定要删除吗？
      </el-form-item>
      <el-form-item>
        <el-row style="width: 100%">
          <el-col :span="16"></el-col>
          <el-col :span="8">
            <el-button @click="deleteConfirmDialogVisible = false;">取消</el-button>
            <el-button type="primary" @click="deleteConfirmDialogVisible = false; deleteSite()">确认</el-button>
          </el-col>
        </el-row>
      </el-form-item>
    </el-form>
  </el-dialog>
  <el-dialog v-model="addSiteDialogVisible" @opened="registerCanvas" @closed="unregisterCanvas">
    <el-form label-width="auto">
      <el-form-item label="场景名">
        <el-input minlength="1" maxlength="30" v-model="newSite.name"></el-input>
      </el-form-item>
      <el-form-item label="在场所内添加房间" label-width="9em">
        <el-switch v-model="hasRoom"></el-switch>
      </el-form-item>
      <!-- TODO: canvas size -->
      <div v-show="!hasRoom">
        <el-row>
          <!-- 绘图工具栏 -->
          <el-col :span="2">
            <el-row>
              <el-color-picker v-model="color" :predefine="predefineColors" />
            </el-row>
            <el-row>
              <el-button style="width: 20px;" @click="undo" :disabled="(undo_stack.length == 1)"><img src="/undo.png" style="width: 15px; height: 15px;"></el-button>
            </el-row>
            <el-row>
              <el-button style="width: 20px;" @click="redo" :disabled="(redo_stack.length == 0)"><img src="/redo.png" style="width: 15px; height: 15px;"></el-button>
            </el-row>
            <el-row>
              <el-button style="width: 20px;"><img src="/reset.png" style="width: 15px; height: 15px;"></el-button>
            </el-row>
            <el-row>
              <el-checkbox-button v-model="isDrawLine" style="width: 20px;" @change="onDrawlineChange">
                <img src="/line.png" style="width: 15px; height: 15px;">
              </el-checkbox-button>
            </el-row>
          </el-col>
          <el-col :span="22">
            <canvas id="new-site-canvas" style="border:1px solid #626364; margin: auto;"></canvas>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="1" :offset="5" style="text-align:center">长</el-col>
          <el-col :span="6">
            <el-input-number :min="MIN_CANVAS_WIDTH" :max="MAX_CANVAS_WIDTH" v-model="canvasWidth"
              @change="canvasSizeChange"></el-input-number>
          </el-col>
          <el-col :span="1" style="text-align:center">宽</el-col>
          <el-col :span="6">
            <el-input-number :min="MIN_CANVAS_HEIGHT" :max="MAX_CANVAS_HEIGHT" v-model="canvasHeight"
              @change="canvasSizeChange"></el-input-number>
          </el-col>
          <el-col :span="1" :offset="1">
            <el-upload v-model:file-list="siteImageFile" action="api/room/image" :on-change="uploadSiteImage"
              :show-file-list="false" :before-upload="checkImageFile">
              <el-button>上传户型图</el-button>
            </el-upload>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="19" :offset="5">修改画布大小和上传图片都会导致画布内容消失，请谨慎操作！</el-col>
        </el-row>
      </div>
      <el-form-item></el-form-item>
      <el-row>
        <el-col :span="6" :offset="6">
          <el-button type="primary" @click="createSite">创建</el-button>
        </el-col>
        <el-col :span="6" :offset="1">
          <el-button @click="addSiteDialogVisible = false">取消</el-button>
        </el-col>
      </el-row>
    </el-form>
  </el-dialog>
  <el-dialog v-model="addRoomDialogVisible">
    <el-form>
      <el-form-item label="房间名">
        <el-input v-model="newRoomName"></el-input>
      </el-form-item>
      <el-row>
        <el-col :offset="8" :span="4"><el-button type="primary" @click="createRoom">创建</el-button></el-col>
        <el-col :span="4"><el-button @click="(addRoomDialogVisible=false)">取消</el-button></el-col>
      </el-row>
    </el-form>
  </el-dialog>
  <el-dialog v-model="deleteRoomDialogVisible">
    删除房间操作不可逆，你确定吗？
    <el-row>
      <el-col :offset="8" :span="4"><el-button type="primary" @click="deleteRoom">确定</el-button></el-col>
      <el-col :span="4"><el-button @click="deleteRoomDialogVisible = false">取消</el-button></el-col>
    </el-row>
  </el-dialog>

  <el-dialog v-model="addDeviceDialogVisible">
    <el-form label-width="auto">
      <el-form-item label="设备名"><el-input v-model="newDeviceName"></el-input></el-form-item>
      <el-form-item label="设备类型">
        <el-select v-model="selectDeviceType" placeholder="请选择">
          <el-option v-for="item in typeOptions" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
    </el-form>
    <el-row>
      <el-col :offset="8" :span="4"><el-button type="primary" @click="addDevice">确定</el-button></el-col>
      <el-col :span="4"><el-button @click="addDeviceDialogVisible = false">取消</el-button></el-col>
    </el-row>
  </el-dialog>
</template>

<script>
// constants
const MOUSE_LEFT_BUTTON = 0
const MIN_CANVAS_WIDTH = 600
const MAX_CANVAS_WIDTH = 800
const MIN_CANVAS_HEIGHT = 150
const MAX_CANVAS_HEIGHT = 500
const UNDO_MAX = 10
const REDO_MAX = 10

// TODO: el-main宽度不变了
// TODO: 当场所数为空时添加场所会默认选择该新增的场所，以及删除场所时会选中下一个场所，这两点应该被修改
// TODO: collapse的开闭似乎由下标决定，所以删除时开闭状态会发生偏移
// TODO: 绘画清空还没做
// TODO: device需要siteID
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import pako from 'pako'
import lightOnUrl from '../../../public/lightOn.png'
import lightOffUrl from '../../../public/lightOff.png'
import switchOnUrl from '../../../public/switchOn.png'
import switchOffUrl from '../../../public/switchOff.png'
import sensorUrl from '../../../public/sensor.png'
import unlockUrl from '../../../public/unlocked.png'
import lockedUrl from '../../../public/locked.png'
// import lightOnUrl from '../../../public/lightOn.png'
// import ConfirmDialog from '../component/ConfirmDialog.vue'
const currentCanvas = ref(null)

const siteImageFile = ref([])
const dialogVisible = ref(false)
const deleteConfirmDialogVisible = ref(false)

const addSiteDialogVisible = ref(false)
const deleteSiteItem = ref(
  {
    name:'',
    index: 0
  }
)
const hasRoom = ref(false)
// current done is stored at the top of undo_stack
const sites = ref([])
const currentSite = ref(null)

const rooms = ref([])
const addRoomDialogVisible = ref(false)
const newRoomName = ref('')
const deleteRoomDialogVisible = ref(false)
const deleteRoomIndex = ref(0)
const currentRoom = ref(null)

const addDeviceDialogVisible = ref(false)
const selectDeviceType = ref(0)
const newDeviceName = ref('')
const isAddDevice = ref(false)
const newDeviceX = ref(0)
const newDeviceY = ref(0)
const deleteDeviceName = ref('')
const deleteDeviceType = ref(0)
let lightOnImage = new Image()
lightOnImage.src = lightOnUrl
let lightOffImage = new Image()
lightOffImage.src = lightOffUrl
let switchOnImage = new Image()
switchOnImage.src = switchOnUrl
let switchOffImage = new Image()
switchOffImage.src = switchOffUrl
let sensorImage = new Image()
sensorImage.src = sensorUrl
let lockedImage = new Image()
lockedImage.src = lockedUrl
let unlockedImage = new Image()
unlockedImage.src = unlockUrl
const deviceImages = {
  lightOn: lightOnImage,
  lightOff: lightOffImage,
  switchOn: switchOnImage,
  switchOff: switchOffImage,
  sensor: sensorImage,
  locked: lockedImage,
  unlocked: unlockedImage
}
const canvasWidth = ref(MIN_CANVAS_WIDTH)
const canvasHeight = ref(MIN_CANVAS_HEIGHT)
const isDrawLine = ref(false)
const undo_stack = ref([])
const redo_stack = ref([])
const color = ref('#000')
const predefineColors = ref([
  '#ff4500',
  '#ff8c00',
  '#ffd700',
  '#90ee90',
  '#00ced1',
  '#1e90ff',
  '#c71585',
])

// drawing
let isDrawing = false
let x = 0
let y = 0
let canvas = null
let ctx = null
let dialog = null
let origin_x = -1
let origin_y = -1
let sitesMap = new Map()
function min(a, b) {
  if (a > b) return b
  else return a
}

function max(a, b) {
  if (a > b) return a
  else return b
}

var checkImageFile = (file) => {
  if (file.size / 1024 / 1024 > 1) {
    ElMessage.error('Image size can not exceed 1MB')
    return false
  }
  else if (file.type != 'image/jpeg' && file.type !== 'image/png') {
    ElMessage.error('Image file must be jpg/png')
    return false
  }
  return true
}

var canvasSizeChange = () => {
  if (canvasHeight.value === undefined || canvasWidth.value === undefined) return
  dialog.style.width = canvasWidth.value + 100 + 'px'
  canvas.width = canvasWidth.value
  canvas.height = canvasHeight.value
}
// TODO: 
var checkSite = () => {
  return true
}

var onDeleteSiteClick = () => {
  let item = document.querySelector(".site-option .is-active")
  if (item) // 选中一个
  {
    deleteConfirmDialogVisible.value = true
    deleteSiteItem.value.name = item.innerHTML
    deleteSiteItem.value.index = parseInt(item.parentElement.id.split('_')[1])
  }
}

var onAddSiteClick = () => {
  addSiteDialogVisible.value = true
}

var onAddDeviceClick = (index, x, y) => {
  addDeviceDialogVisible.value = true
  if (index < 0) currentRoom.value = null
  else currentRoom.value = rooms.value[index]
  newDeviceX.value = x
  newDeviceY.value = y
  console.log(currentRoom.value)
}

function onDeleteDeviceClick(roomIndex, deviceIndex){
  var device = rooms.value[roomIndex].devices[deviceIndex]
  this.$axios.post('device/delete', {name: device.name, type: device.type, roomName: device.roomName, siteName: device.siteName, token: sessionStorage.getItem('token')})
    .then(response => {
      if (response.data.code == 200) {
        ElMessage.success("成功删除设备" + device.name)
        rooms.value[roomIndex].devices.splice(deviceIndex, 1)
      }
      else {
        ElMessage.error(response.data.message)
      }
    })
}

function onDeleteSiteDeviceClick(){
  this.$axios.post('device/delete', {name: deleteDeviceName.value, type: deleteDeviceType.value, roomName: null, siteName: currentSite.value.name, token: sessionStorage.getItem('token')})
    .then(response => {
      if (response.data.code == 200) {
        ElMessage.success("成功删除设备" + deleteDeviceName.value)
        try {
          currentSite.value.devices.forEach((device, index) => {
            if (device.name == deleteDeviceName.value && device.type == deleteDeviceType.value) {
              console.log('find')
              currentSite.value.devices.splice(index, 1)
              throw new Error('LoopInterrupt')
            }
          })
        }
        catch(e){}
      }
      else {
        ElMessage.error(response.data.message)
      }
    })
}

var uploadSiteImage = (file, fileList) => {
  // TODO: repetitively get canvas ctx
  // const canvas = document.getElementById("new-site-canvas")
  const image = new Image()
  if (fileList.length > 1) {
    fileList.splice(0, 1)
  }
  siteImageFile.value = fileList
  image.src = URL.createObjectURL(siteImageFile.value[0].raw)
  image.onload = function () {
    URL.revokeObjectURL(this.src)
    // 直接拉伸图片到符合画布
    let scale_x = canvas.width / image.width
    let scale_y = canvas.height / image.height
    ctx.width = image.width * scale_x
    ctx.height = image.height * scale_y
    ctx.drawImage(image, 0, 0, ctx.width, ctx.height)
  }
}

var registerCanvas = () => {
  // 找到canvas和dialog对象
  canvas = document.getElementById("new-site-canvas")
  ctx = canvas.getContext("2d")
  dialog = document.querySelector('.el-dialog')

  // 初始化dialog和canvas的宽度
  canvas.width = min(max(canvas.parentElement.offsetWidth - 10, MIN_CANVAS_WIDTH), MAX_CANVAS_WIDTH)
  canvas.height = MIN_CANVAS_HEIGHT
  canvasWidth.value = canvas.width
  canvasHeight.value = canvas.height
  dialog.style.width = canvasWidth.value + 100 + 'px'

  // 初始化undo和redo堆栈
  undo_stack.value = [ctx.getImageData(0, 0, canvas.width, canvas.height)]
  redo_stack.value = []

  window.timer = setInterval(
    () => {
      dialog.style.width = canvasWidth.value + 100 + 'px'
    },
    1000
  )

  // ---------- Drawing -------------
  // TODO: delete listener
  canvas.addEventListener("mousedown", (event) => {
    if (event.button !== MOUSE_LEFT_BUTTON) {
      return;
    }

    x = event.offsetX;
    y = event.offsetY;
    isDrawing = true;
  })

  canvas.addEventListener("mousemove", (event) => {
    if (isDrawLine.value) {
      if (!(origin_x == -1 && origin_y == -1))
      {
        ctx.putImageData(undo_stack.value.at(-1), 0, 0)
        drawLine(ctx, color.value, 1, origin_x, origin_y, event.offsetX, event.offsetY)
      }
    }
    else if (isDrawing) {
      drawLine(ctx, color.value, 1, x, y, event.offsetX, event.offsetY)
    }
    x = event.offsetX;
    y = event.offsetY;
  })

  canvas.addEventListener("mouseup", (event) => {
    if (event.button !== MOUSE_LEFT_BUTTON) {
      return;
    }
    if (isDrawLine.value) {
      if (!(origin_x == -1 && origin_y == -1)) {
        drawLine(ctx, color.value, 1, origin_x, origin_y, event.offsetX, event.offsetY)
      }

      // 如果画了一条直线
      if (!(origin_x == -1 && origin_y == -1)) {
        if (undo_stack.value.length == UNDO_MAX)
        {
          undo_stack.value.shift()
        }
        undo_stack.value.push(ctx.getImageData(0, 0, canvas.width, canvas.height))
        redo_stack.value = [] // 一旦有绘制，清空redo_stack
      }
      origin_x = x
      origin_y = y
      isDrawing = false
      return
    }

    if (isDrawing) {
      drawLine(ctx, color.value, 1, x, y, event.offsetX, event.offsetY);
      x = 0;
      y = 0;
      // 结束画曲线
      isDrawing = false;
      if (undo_stack.value.length == UNDO_MAX)
      {
        undo_stack.value.shift()
      }
      undo_stack.value.push(ctx.getImageData(0, 0, canvas.width, canvas.height))
      redo_stack.value = [] // 一旦有绘制，清空redo_stack
    }
  })
}

var onDrawlineChange = (checked) => {
  isDrawing = false
  if (!checked) ctx.putImageData(undo_stack.value.at(-1), 0, 0)
  else {
    origin_x = -1
    origin_y = -1
  }
}

var unregisterCanvas = () => {
  clearInterval(window.timer)
  canvas = ctx = dialog = null
}

var refreshCurrentCanvas = () => {
  const ctx = currentCanvas.value.getContext('2d')
  ctx.putImageData(currentSite.value.image, 0, 0)
  var image
  currentSite.value.devices.forEach((device) => {
    if (device.type == 0) {
      if (device.value == '1') image = deviceImages.lightOn
      else image = deviceImages.lightOff
    }
    else if (device.type == 1) {
      if (device.value == '1') image = deviceImages.switchOn
      else image = deviceImages.switchOff
    }
    else if (device.type == 2) {
      image = deviceImages.sensor
    }
    else if (device.type == 3) {
      if (device.value == '1') image = deviceImages.locked
      else image = deviceImages.unlocked
    }
    ctx.drawImage(image, device.x, device.y)
    ctx.fillText(device.name, device.x + 24, device.y + 12)
    if (device.type == 2 && device.value != null) ctx.fillText(device.value, device.x + 24, device.y + 24)
  })
}

var undo = () => {
  if (undo_stack.value.length > 1) {
    if (redo_stack.value.length == REDO_MAX) {
      redo_stack.value.shift()
    }
    redo_stack.value.push(undo_stack.value.pop())
    ctx.putImageData(undo_stack.value.at(-1), 0, 0)
    origin_x = -1
    origin_y = -1
  }
}

var redo = () => {
  if (redo_stack.value.length != 0) {
    if (undo_stack.value.length == UNDO_MAX) {
      undo_stack.value.shift()
    }
    let tmp = redo_stack.value.pop()
    ctx.putImageData(tmp, 0, 0)
    undo_stack.value.push(tmp)
  }
}

function drawLine(context, strokeStyle, lineWidth, x0, y0, x1, y1) {
  context.beginPath();
  context.strokeStyle = strokeStyle;
  context.lineWidth = lineWidth;
  context.moveTo(x0, y0);
  context.lineTo(x1, y1);
  context.stroke();
}

export default {
  name: 'AppIndex',
  setup() {
    const side_bar_width = 400
    const mainWidth = ref(window.outerWidth - side_bar_width)
    window.addEventListener('resize', () => {
      mainWidth.value = window.outerWidth - side_bar_width
    })
    return { mainWidth }
  },
  mounted() {
    let _this = this
    function getSites(){
      var token = sessionStorage.getItem('token')
      _this.$axios.post('site', {token: token}).then(successResponse => {
        sites.value = []
        successResponse.data.forEach((item, index, arr) => {
          let imageData = item.image === null ? null :
              new ImageData(Uint8ClampedArray.from(pako.inflate(Uint8Array.from(item.image))), item.width, item.height)
          sites.value.push({
            name: item.name,
            hasRoom: item.hasRoom,
            image: imageData,
            index: '2-' + (index + 1)
          })
        })
      })
    }
    sites.value = getSites()
  },
  data() {
    return {
      currentCanvas,
      dialogVisible,
      deleteConfirmDialogVisible,
      addSiteDialogVisible,
      addRoomDialogVisible,
      deleteRoomDialogVisible,
      siteImageFile,
      hasRoom,
      color,
      predefineColors,
      canvasWidth,
      canvasHeight,
      MIN_CANVAS_WIDTH,
      MAX_CANVAS_WIDTH,
      MIN_CANVAS_HEIGHT,
      MAX_CANVAS_HEIGHT,
      isDrawLine,
      undo_stack,
      redo_stack,
      rooms,
      sites,
      currentSite,
      newSite: {
        name: '',
        image: ''
      },
      newRoomName,
      deleteRoomIndex,
      addDeviceDialogVisible,
      selectDeviceType,
      newDeviceName,
      deleteDeviceName,
      deleteDeviceType,
      typeOptions:[
        {
          label: "灯",
          value: 0
        },
        {
          label: "开关",
          value: 1
        },
        {
          label: "传感器",
          value: 2
        },
        {
          label: "门锁",
          value: 3
        }
      ],
      isAddDevice
    }
  },
  methods: {
    onDeleteSiteClick,
    onAddSiteClick,
    onAddDeviceClick,
    uploadSiteImage,
    checkImageFile,
    checkSite,
    registerCanvas,
    unregisterCanvas,
    canvasSizeChange,
    onDrawlineChange,
    undo,
    redo,
    onDeleteDeviceClick,
    onDeleteSiteDeviceClick,
    deleteRoom () {
      this.$axios.post('room/delete', {name: rooms.value[deleteRoomIndex.value].name, siteName: currentSite.value.name, token: sessionStorage.getItem('token')}).then(
        successResponse => {
          if (successResponse.data.code == 200)
          {
            ElMessage.success("成功删除房间")
            rooms.value.splice(deleteRoomIndex, 1)
            deleteRoomDialogVisible.value = false
          }
          else
          {
            ElMessage.error(successResponse.data.code)
          }
        }
      ) 
    },
    handleSelect(index, indexPath){
      let arr = index.split()
      if (index[0] == '1')
      {

      }
      else if (index[0] != '2')
      {
        // TODO: 可以考虑点击后再读取图片，提高性能
        let idx = parseInt(index.split('_')[1])
        console.log(sites.value[idx])
        currentSite.value = sites.value[idx]
        if (currentSite.value.hasRoom) {
          rooms.value = sitesMap.get(currentSite.value.name)
          if (rooms.value === undefined) { // 还没有请求过该场所的房间
            console.log(currentSite.value.name)
            let _this = this
            _this.$axios.post('room', {name: currentSite.value.name, token: sessionStorage.getItem('token')})
              .then(successResponse => {
                rooms.value = successResponse.data
                console.log(rooms.value)
                rooms.value.forEach((room) => {
                  this.$axios.post('device', {name: room.name, siteName: currentSite.value.name, token: sessionStorage.getItem('token')})
                    .then(response => {
                      var devices = response.data
                      console.log(devices)
                      devices.forEach((device) => {
                        device.roomName = room.name
                        device.siteName = currentSite.value.name
                        device['value'] = 0
                      })
                      room['devices'] = devices
                    })
                })
                console.log(rooms.value)
                sitesMap.set(currentSite.value.name, rooms.value) 
              })
          }
        }
        else
        {
          this.$axios.post('device', {name: null, siteName: currentSite.value.name, token: sessionStorage.getItem('token')})
            .then(response => {
              var devices = response.data
              console.log(devices)
              devices.forEach((device) => {
                device.roomName = null
                device.siteName = currentSite.value.name
                device['value'] = 0
              })
              currentSite.value['devices'] = devices
            })
          this.$nextTick(()=> {
            currentCanvas.value = document.querySelector('#_site' + currentSite.value.name)
            if (currentCanvas.value === null) {
              ElMessage.error('数据异常')
            }
            else
            {
              const ctx = currentCanvas.value.getContext('2d')
              currentCanvas.value.width = currentSite.value.image.width
              currentCanvas.value.height = currentSite.value.image.height
              ctx.putImageData(currentSite.value.image, 0, 0)
              currentCanvas.value.addEventListener("mouseup", (event) => {
                if (event.button != MOUSE_LEFT_BUTTON) return
                if (isAddDevice.value) {
                  onAddDeviceClick(-1, event.offsetX, event.offsetY)
                }
              })
              window.timer = setInterval(refreshCurrentCanvas, 1000)
            }
          })
        }
      }
    },
    deleteSite() {
      this.$axios.post('site/delete', { name: deleteSiteItem.value.name, token: sessionStorage.getItem('token')}).then(
        successResponse => {
          if (successResponse.data.code == 200)
          {
            sites.value.splice(deleteSiteItem.value.index, 1)
            sitesMap.delete(deleteSiteItem.value.name)
            currentSite.value = null
          }
          else
          {
            ElMessage.error(successResponse.data.message)
          }
        }
      )
    },
    createSite() {
      if (checkSite()) {
        if (this.newSite.name === undefined || this.newSite.name === "")
        {
          ElMessage.error("场景名不得为空")
          return
        }
        let tmp = undo_stack.value.at(-1)
        this.$axios.post('site/new', 
          { 
            name : this.newSite.name, 
            hasRoom: hasRoom.value,
            image: Array.from(pako.deflate(Uint8Array.from(tmp.data))), // Uint8ClampedArray、Uint8Array cannot be parsed by Java, Use pako to compress data
            width: tmp.width,
            height: tmp.height,
            token: sessionStorage.getItem('token')
          })
          .then(successResponse => {
            if (successResponse.data.code == 200) {
              ElMessage.success('成功创建')
              var site = {
                name: this.newSite.name,
                hasRoom: hasRoom.value,
                image: tmp,
                index: '2-' + (sites.value.at(-1)?.index == undefined ? 1 : sites.value.at(-1)?.index + 1)
              }
              if (!hasRoom.value) site['devices'] = []
              console.log(site)
              sites.value.push(site)
              addSiteDialogVisible.value = false
            }
            else {
              ElMessage.error(successResponse.data.message)
            }
          })
      }
    },
    createRoom() {
      var room = {name: newRoomName.value, siteName: currentSite.value.name, token: sessionStorage.getItem('token')}
      this.$axios.post('room/new', room)
        .then(successResponse => {
          if (successResponse.data.code == 200) {
            ElMessage.success("创建成功")
            addRoomDialogVisible.value = false
            newRoomName.value = ''
            rooms.value.push({name: room.name, siteName: room.siteName, devices: []})
          }
          else {
            ElMessage.error(successResponse.data.message)
          }
        })
    },
    addDevice() {
      var roomName = currentRoom.value == null ? null : currentRoom.value.name
      var device = {name: newDeviceName.value, type: selectDeviceType.value, roomName: roomName, siteName: currentSite.value.name, 
                      x: newDeviceX.value, y: newDeviceY.value, token: sessionStorage.getItem('token')}
      console.log(device)
      this.$axios.post('device/new', device)
        .then(successResponse => {
          if (successResponse.data.code == 200) {
            ElMessage.success("创建成功")
            addDeviceDialogVisible.value = false
            newDeviceName.value = ''
            if (currentSite.value.hasRoom)
              currentRoom.value.devices.push({name : device.name, type: device.type, value: 0})
            else
              currentSite.value.devices.push({name : device.name, type: device.type, value: 0, x: device.x, y: device.y})
            console.log(currentRoom.value)
            console.log(rooms.value)
          }
          else {
            ElMessage.error(successResponse.data.message)
          }
        })
    }
  },
  computed: {
    checkHasRoom() {
      return currentSite.value.hasRoom
    },
    currentSiteCanvas() {
      return "_site" + currentSite.value.name
    },
    siteIndex() {
      return function(index){
        return "site_" + index
      }
    },
    roomIndex() {
      return function(index) {
        return "room_" + index
      }
    },
    deviceValueMap() {
      return function(device) {
        if (device.type == 0 || device.type == 1) {
          if (device.value == 0) return "关"
          else if (device.value == 1) return "开"
        }
        else if (device.type == 2) {
          if (device.value == null) return '暂无数据'
        }
        else if (device.type == 3) {
          if (device.value == 1) return '已上锁'
          else if (device.value == 0) return "未上锁"
        }
        console.log(device)
        return '1'
      }
    }
  }
  // components: {
  //   ConfirmDialog
  // }
}
</script>

<style scoped>
.white {
  color: white;
}

.el-header {
  margin-top: 10px;
  padding-left: 10px;
  text-align: left;
  width: 100%;
  height: 80px;
  border-bottom: 1px solid #626364;
}

.circle-button {
  color: white;
  background-color: #252525;
  border-radius: 80px;
  border: 0px;
  width: 30px;
}

.circle-button:hover {
  background-color: #256455;
}

.side-bar-scroll {
  padding-top: 20px;
  padding-bottom: 100px;
  position: fixed;
  height: 100%;
  background-color: #252525;
  border-right: 1px solid #626364;
}

.side-bar {
  width: 200px;
  height: 100%;
}

/* .el-main {
  width: 100%;
} */
.main-bar-scroll {
  padding-right: 20px;
  padding-bottom: 100px;
  position: fixed;
  height: 100%;
  width: 100%;
  /* width: var(--main_width); */
  background-color: #252525;
}

.is-active {
  background-color: #256455;
  border-radius: 30px;
}

.el-menu-item:hover {
  background-color: #00463d;
  border-radius: 23px;
}

.el-menu-item {
  min-width: 200px;
  border-radius: 23px;
}

</style>

<style>
.is-opened .el-sub-menu__title {
  color: #04ccaf !important;
}

.el-menu {
  border: 0;
}

.el-collapse {
  border-top: 0px;
  border-bottom: 0px;
}

.el-collapse-item__header {
  padding-left: 30px;
  border-top: 1px solid #626364;
  border-bottom: 1px solid #626364;
  background-color: #252525;
  color: white;
  font: 1em sans-serif;
}

.el-checkbox-button__inner {
  padding-left: 8px;
  padding-right: 8px;
}

</style>