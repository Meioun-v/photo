<template>
  <view class="app-container">
    <!-- 左侧预览栏 -->
    <view class="preview-sidebar" v-if="showThumbnail">
      <view 
        v-for="i in totalPhotoCount" 
        :key="i" 
        class="preview-item"
        :class="{'current': i === currentPhotoIndex}"
      >
        <view class="preview-number">{{i}}</view>
        <view class="empty" v-if="!photos[i-1]">
          <uni-icons type="camera" size="24" color="rgba(255, 255, 255, 0.7)"></uni-icons>
        </view>
        <image v-else :src="photos[i-1].local_path" mode="aspectFill"></image>
      </view>
    </view>
    
    <!-- 相机视图 -->
    <view class="camera-view">
      <!-- 左上角多功能按钮 -->
      <view class="corner-btn left-corner" @click="handleLeftCornerClick">
        <uni-icons :type="leftCornerIcon" size="32" color="#ffffff"></uni-icons>
        <text class="corner-text">{{leftCornerText}}</text>
      </view>
      
      <!-- 右上角多功能按钮 -->
      <view class="corner-btn right-corner" @click="handleRightCornerClick">
        <uni-icons :type="rightCornerIcon" size="32" color="#ffffff"></uni-icons>
        <text class="corner-text">{{rightCornerText}}</text>
      </view>
      
      <view class="camera-container" :class="{ 'flash': isFlashing }">
        <camera 
          v-if="!isElectron && !isH5 && canUseCamera"
          :device-position="devicePosition" 
          :flash="flashEnabled ? 'on' : 'off'" 
          @error="handleCameraError"
          class="camera"
        ></camera>
        
        <!-- Web相机代替方案将由JS动态添加 -->
        <view v-if="(isElectron || isH5) && !videoElement" class="camera web-camera-placeholder"></view>
        
        <!-- 相机错误显示 -->
        <view v-if="!canUseCamera" class="camera-error">
          <view class="error-icon">
            <uni-icons type="closeempty" size="48" color="#ffffff"></uni-icons>
          </view>
          <view class="error-message">
            {{ errorMessage || '无法访问相机，请检查权限设置或尝试刷新页面' }}
          </view>
          <view class="action-btn retry-btn" @click="retryCamera" style="margin-top: 20px;">
            <uni-icons type="reload" size="18" color="#ffffff"></uni-icons>
            <text>重试</text>
          </view>
        </view>
        
        <!-- 倒计时 -->
        <view class="countdown" v-if="countdown > 0">
          <text class="countdown-number">{{ countdown }}</text>
        </view>
        
        <!-- 闪光效果 -->
        <view class="flash-overlay" :class="{'flash-active': isFlashing}"></view>
        
        <!-- 显示已拍照片 -->
        <image 
          v-if="lastPhotoPath && !isCounting" 
          :src="lastPhotoPath" 
          class="captured-photo-display"
          mode="aspectFill"
        ></image>
      </view>
      
      <!-- 照片滤镜选择器 -->
      <!-- 
      <view class="filter-selector" v-if="!lastPhotoPath && !isCounting">
        <scroll-view scroll-x class="filter-scroll">
          <view 
            v-for="filter in filterOptions" 
            :key="filter.id" 
            class="filter-option"
            :class="{ 'active': selectedFilter && selectedFilter.id === filter.id }"
            @click="selectFilter(filter)"
          >
            <view class="filter-preview" :style="{ filter: filter.filter }">
              <uni-icons v-if="selectedFilter && selectedFilter.id === filter.id" type="checkmarkempty" size="20" color="#ffffff"></uni-icons>
            </view>
            <view class="filter-name">{{ filter.name }}</view>
          </view>
        </scroll-view>
      </view>
      -->
      
      <!-- 底部控制按钮 -->
      <!-- <view class="bottom-controls" v-if="!lastPhotoPath && !isCounting && (isElectron || isH5)">
        <view class="control-btn" @click="toggleCameraPosition">
          <uni-icons type="loop" size="24" color="#ffffff"></uni-icons>
        </view> -->
        
        <!-- 添加开始拍照按钮 -->
        <!-- <view class="control-btn start-capture-btn" @click="startCapture" v-if="!isCounting && !isCapturing">
          <uni-icons type="camera-filled" size="24" color="#ffffff"></uni-icons>
        </view>
         -->
        <!-- Electron开发环境中添加测试图片按钮 -->
        <!-- <view class="control-btn" @click="useTestImage" v-if="isElectron">
          <uni-icons type="image" size="24" color="#ffffff"></uni-icons>
        </view>
      </view> -->
      
      <!-- 调试信息显示 -->
      <view class="debug-info" v-if="isElectron">
        <view>平台: {{ isElectron ? 'Electron' : isH5 ? 'Web' : 'Native' }}</view>
        <view>相机状态: {{ canUseCamera ? '可用' : '不可用' }}</view>
        <view v-if="!canUseCamera">错误: {{ errorMessage }}</view>
        <view>视频元素: {{ videoElement ? '已创建' : '未创建' }}</view>
      </view>
    </view>
    
    <!-- 在页面某个位置（比如你想隐藏但保留功能，可以不添加此UI部分） -->
    <view class="beauty-switch" v-if="false">
      <text>自动美颜</text>
      <switch :checked="autoBeautyEnabled" @change="toggleAutoBeauty" color="#1a73e8" />
    </view>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'

export default {
  components: {
    uniIcons
  },
  data() {
    return {
      templateId: '',
      totalPhotoCount: 2, // 默认4张照片
      currentPhotoIndex: 1,
      countdown: 0,
      isCounting: false,
      isCapturing: false,
      isFlashing: false,
      lastPhotoPath: '',
      photos: [], // 存储照片信息的数组
      cameraContext: null,
      countdownSeconds: 5,
      flashScreen: true,
      devicePosition: 'front',
      flashEnabled: false,
      selectedTemplate: null,
      tempPhotoPath: null, // 临时存储当前拍摄的照片
      isElectron: false,  // 是否运行在Electron环境
      isH5: false,       // 是否运行在H5环境
      videoElement: null, // 原生video元素引用
      stream: null,      // 媒体流
      errorMessage: '',  // 错误信息
      canUseCamera: true, // 是否可以使用相机
      debugMode: true,   // 调试模式
      testImages: [      // 测试图片
        '/static/test_photo_1.jpg',
        '/static/test_photo_2.jpg',
        '/static/test_photo_3.jpg',
        '/static/test_photo_4.jpg'
      ],
      showThumbnail: true, // 默认显示缩略图
      
      // 照片滤镜效果 - 添加滤镜选项
      filterOptions: [
        { id: 'normal', name: '原图', filter: '' },
        { id: 'warm', name: '暖色', filter: 'saturate(1.3) sepia(0.2)' },
        { id: 'cool', name: '冷色', filter: 'saturate(1.1) hue-rotate(180deg)' },
        { id: 'bw', name: '黑白', filter: 'grayscale(1.1)' },
        { id: 'vintage', name: '复古', filter: 'sepia(0.5) contrast(1.2)' }
      ],
      selectedFilter: null, // 选中的滤镜
      leftCornerIcon: 'back',
      rightCornerIcon: 'camera-filled',
      leftCornerText: '返回',
      rightCornerText: '开始拍摄',
      
      // 自动美颜相关
      autoBeautyEnabled: true,  // 默认启用自动美颜
    }
  },
  computed: {
    hasCompletedPhotos() {
      // 检查是否至少有一张照片已完成
      return this.photos.some(photo => photo !== null);
    },
    platformInfo() {
      // 返回平台信息用于调试
      return {
        isElectron: this.isElectron,
        isH5: this.isH5,
        canUseCamera: this.canUseCamera
      };
    }
  },
  onLoad(options) {
    this.templateId = options.template_id || '';
    this.totalPhotoCount = parseInt(options.photo_count || 4);
    
    // 初始化照片数组
    this.photos = new Array(this.totalPhotoCount).fill(null);
    
    // 获取选中的模板
    const template = uni.getStorageSync('selected_template');
    if (template) {
      this.selectedTemplate = template;
      this.totalPhotoCount = template.photo_count || this.totalPhotoCount;
    }
    
    // 加载捕捉设置
    this.loadCaptureSettings();
    
    // 检测运行环境
    this.detectPlatform();
    
    // 设置默认滤镜
    this.selectedFilter = this.filterOptions[0];
    
    // 初始化相机
    this.initCamera();
  },
  onUnload() {
    // 停止媒体流
    this.stopMediaStream();
  },
  methods: {
    // 加载捕捉设置
    loadCaptureSettings() {
      try {
        const cachedSettings = localStorage.getItem('app_settings');
        if (cachedSettings) {
          const settings = JSON.parse(cachedSettings);
          
          if (settings.capture && settings.capture.length > 0) {
            // 加载倒计时秒数
            const countdown = settings.capture.find(s => s.key === 'capture_countdown_seconds');
            if (countdown) {
              let value = Number(countdown.value);
              if (!isNaN(value) && value >= 3 && value <= 8) {
                this.countdownSeconds = value;
              }
            }
            
            // 加载闪光效果设置
            const flash = settings.capture.find(s => s.key === 'capture_flash_screen');
            if (flash !== undefined) {
              this.flashScreen = flash.value === true;
            }
            
            // 加载缩略图显示设置
            const thumbnail = settings.capture.find(s => s.key === 'capture_thumbnail_show');
            if (thumbnail !== undefined) {
              this.showThumbnail = thumbnail.value === true;
            } else {
              this.showThumbnail = true; // 默认显示
            }
            
            console.log('已加载捕捉设置:', {
              countdownSeconds: this.countdownSeconds,
              flashScreen: this.flashScreen,
              showThumbnail: this.showThumbnail
            });
          }
        }
      } catch (error) {
        console.error('加载捕捉设置失败:', error);
      }
    },
    
    // 检测当前运行平台
    detectPlatform() {
      // 检查是否在Electron环境中运行
      this.isElectron = window && window.process && window.process.type === 'renderer';
      
      // 检查是否在H5环境中运行
      // #ifdef H5
      this.isH5 = true;
      // #endif
      
      console.log('平台检测结果:', this.platformInfo);
    },
    
    // 初始化相机
    async initCamera() {
      try {
        // 如果在Electron或H5环境中，使用Web API
        if (this.isElectron || this.isH5) {
          await this.initWebCamera();
        } else {
          // 在原生环境中使用uni-app API
          try {
            this.cameraContext = uni.createCameraContext();
          } catch (e) {
            console.error('创建相机上下文失败:', e);
            this.errorMessage = '无法创建相机上下文';
            this.canUseCamera = false;
            
            // 尝试回退到Web API
            await this.initWebCamera();
          }
        }
      } catch (e) {
        console.error('初始化相机失败:', e);
        this.handleCameraError({
          detail: { errMsg: e.message || '初始化相机失败' }
        });
        this.canUseCamera = false;
      }
    },
    
    // 使用Web API初始化相机 - 根据Camera.vue的实现逻辑更新
    async initWebCamera() {
      if (navigator && navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        try {
          // 获取所有可用的视频设备
          const devices = await navigator.mediaDevices.enumerateDevices();
          const videoDevices = devices.filter(device => device.kind === 'videoinput');
          
          // 相机类型选择
          const facingMode = this.devicePosition === 'front' ? 'user' : 'environment';
          
          // 构建视频约束
          const videoConstraints = {
            facingMode,
            width: { ideal: 1920 },
            height: { ideal: 1080 }
          };
          
          // 如果有多个摄像头，尝试选择高分辨率摄像头
          if (videoDevices.length > 1) {
            try {
              // 获取每个摄像头的详细信息
              const deviceCapabilities = await Promise.all(
                videoDevices.map(async (device) => {
                  try {
                    const stream = await navigator.mediaDevices.getUserMedia({
                      video: {
                        deviceId: { exact: device.deviceId },
                        width: { ideal: 1920 },
                        height: { ideal: 1080 }
                      }
                    });
                    const track = stream.getVideoTracks()[0];
                    const capabilities = track.getCapabilities();
                    stream.getTracks().forEach(track => track.stop());
                    return {
                      deviceId: device.deviceId,
                      label: device.label,
                      capabilities: capabilities
                    };
                  } catch (e) {
                    console.warn(`无法获取设备 ${device.deviceId} 的详细信息:`, e);
                    return null;
                  }
                })
              );
              
              // 过滤掉无效的设备信息
              const validDevices = deviceCapabilities.filter(device => device !== null);
              
              // 按分辨率排序设备
              validDevices.sort((a, b) => {
                const aRes = (a.capabilities.width?.max || 0) * (a.capabilities.height?.max || 0);
                const bRes = (b.capabilities.width?.max || 0) * (b.capabilities.height?.max || 0);
                return bRes - aRes;
              });
              
              // 选择分辨率最高的设备
              if (validDevices.length > 0) {
                const bestDevice = validDevices[0];
                console.log('选择最高分辨率摄像头:', bestDevice);
                videoConstraints.deviceId = { exact: bestDevice.deviceId };
              }
            } catch (e) {
              console.warn('获取摄像头详细信息失败，使用默认设置:', e);
            }
          }

          // 临时使用前置摄像头 HD Webcam
          // videoConstraints.deviceId = { exact: "dc789883712ea6943b3fa3b89eeefc49941eacfceb959bf8893cba2374ab6878" };
          
          // 请求相机权限和视频流
          this.stream = await navigator.mediaDevices.getUserMedia({ 
            video: videoConstraints,
            audio: false
          });
          
          // 延迟执行，确保DOM已更新
          setTimeout(() => {
            // 创建或获取video元素
            let videoEl = document.querySelector('.web-camera');
            if (!videoEl) {
              // 如果没有找到video元素，创建一个
              const newVideo = document.createElement('video');
              newVideo.className = 'web-camera';
              newVideo.autoplay = true;
              newVideo.playsInline = true;
              
              // 添加样式
              newVideo.style.width = '100%';
              newVideo.style.height = '100%';
              newVideo.style.objectFit = 'cover';
              
              // 如果有滤镜，应用滤镜
              if (this.selectedFilter && this.selectedFilter.filter) {
                newVideo.style.filter = this.selectedFilter.filter;
              }
              
              // 将video添加到相机容器
              const container = document.querySelector('.camera-container');
              if (container) {
                container.appendChild(newVideo);
                this.videoElement = newVideo;
              }
            } else {
              this.videoElement = videoEl;
              
              // 如果有滤镜，应用滤镜
              if (this.selectedFilter && this.selectedFilter.filter) {
                this.videoElement.style.filter = this.selectedFilter.filter;
              }
            }
            
            // 设置视频源
            if (this.videoElement) {
              this.videoElement.srcObject = this.stream;
              
              // 设置metadata加载回调获取实际分辨率
              this.videoElement.onloadedmetadata = () => {
                this.videoElement.play();
                console.log('Web相机已初始化，分辨率:', 
                  `${this.videoElement.videoWidth}x${this.videoElement.videoHeight}`);
                
                // 更新相机状态信息
                if (this.videoElement.videoWidth && this.videoElement.videoHeight) {
                  // 这里可以添加相机状态更新的逻辑
                }
              };
            }
          }, 500);
          
          this.canUseCamera = true;
        } catch (e) {
          console.error('获取媒体流失败:', e);
          this.errorMessage = '无法访问相机: ' + (e.message || '未知错误');
          this.canUseCamera = false;
        }
      } else {
        console.error('浏览器不支持getUserMedia API');
        this.errorMessage = '浏览器不支持相机功能';
        this.canUseCamera = false;
      }
    },
    
    // 停止媒体流
    stopMediaStream() {
      if (this.stream) {
        const tracks = this.stream.getTracks();
        tracks.forEach(track => track.stop());
        this.stream = null;
        
        if (this.videoElement) {
          this.videoElement.srcObject = null;
        }
      }
    },
    
    startCapture() {
      this.isCapturing = true;
      this.leftCornerIcon = 'closeempty';
      this.leftCornerText = '取消拍摄';
      this.rightCornerIcon = 'checkmarkempty';
      this.rightCornerText = '确认拍摄';
      this.startCountdown();
    },
    
    async takePhoto() {
      this.isCapturing = true;
      
      // 闪屏效果
      if (this.flashScreen) {
        console.log('应用闪光效果');
        this.isFlashing = true;
        
        // 延迟拍照，先让闪光效果显示
        await new Promise(resolve => setTimeout(resolve, 100));
        
        // 拍照
        try {
          let result;
          
          // 使用Web API或原生API拍照
          if ((this.isElectron || this.isH5) && this.videoElement) {
            result = await this.takePhotoWithWebAPI();
          } else if (this.cameraContext) {
            result = await this.takePhotoWithUniApp();
          } else {
            throw new Error('无可用的相机API');
          }
          
          // 保存照片信息
          this.lastPhotoPath = result.tempImagePath || result.path;
          this.tempPhotoPath = result.tempImagePath || result.path;
          
          // 延迟关闭闪光效果，使其更明显
          setTimeout(() => {
            this.isFlashing = false;
          }, 300);
          
          // 显示操作按钮，等待用户确认
          this.isCapturing = false;
        } catch (e) {
          this.isFlashing = false;
          uni.showToast({
            title: '拍照失败: ' + (e.message || '未知错误'),
            icon: 'none'
          });
          this.isCapturing = false;
          console.error('拍照失败:', e);
        }
      } else {
        console.log('闪光效果已禁用');
        // 不使用闪光效果，直接拍照
        try {
          let result;
          
          // 使用Web API或原生API拍照
          if ((this.isElectron || this.isH5) && this.videoElement) {
            result = await this.takePhotoWithWebAPI();
          } else if (this.cameraContext) {
            result = await this.takePhotoWithUniApp();
          } else {
            throw new Error('无可用的相机API');
          }
          
          // 保存照片信息
          this.lastPhotoPath = result.tempImagePath || result.path;
          this.tempPhotoPath = result.tempImagePath || result.path;
          
          // 显示操作按钮，等待用户确认
          this.isCapturing = false;
        } catch (e) {
          uni.showToast({
            title: '拍照失败: ' + (e.message || '未知错误'),
            icon: 'none'
          });
          this.isCapturing = false;
          console.error('拍照失败:', e);
        }
      }
    },
    
    // 使用uni-app API拍照
    async takePhotoWithUniApp() {
      return new Promise((resolve, reject) => {
        if (!this.cameraContext) {
          reject(new Error('相机上下文不存在'));
          return;
        }
        
        this.cameraContext.takePhoto({
          quality: 'high',
          success: resolve,
          fail: reject
        });
      });
    },
    
    
    // 使用Web API拍照 - 根据Camera.vue的实现逻辑更新
    async takePhotoWithWebAPI() {
      return new Promise((resolve, reject) => {
        try {
          if (!this.videoElement) {
            reject(new Error('视频元素不存在'));
            return;
          }
    
          // 获取视频元素的实际显示尺寸
          const videoRect = this.videoElement.getBoundingClientRect();
          const videoDisplayWidth = videoRect.width;
          const videoDisplayHeight = videoRect.height;
    
          // 计算视频实际内容的缩放比例
          const scaleX = videoDisplayWidth / this.videoElement.videoWidth;
          const scaleY = videoDisplayHeight / this.videoElement.videoHeight;
          const scale = Math.max(scaleX, scaleY);
    
          // 计算视频实际内容的偏移量
          const offsetX = (videoDisplayWidth - this.videoElement.videoWidth * scale) / 2;
          const offsetY = (videoDisplayHeight - this.videoElement.videoHeight * scale) / 2;
    
          // 创建canvas用于截图
          const canvas = document.createElement('canvas');
          const ctx = canvas.getContext('2d');
    
          // 设置canvas尺寸为视频显示尺寸
          canvas.width = videoDisplayWidth;
          canvas.height = videoDisplayHeight;
    
          // 将视频帧绘制到canvas
          ctx.drawImage(
            this.videoElement,
            0, 0, this.videoElement.videoWidth, this.videoElement.videoHeight,
            offsetX, offsetY, this.videoElement.videoWidth * scale, this.videoElement.videoHeight * scale
          );
    
          // 应用滤镜效果
          if (this.selectedFilter && this.selectedFilter.filter) {
            ctx.filter = this.selectedFilter.filter;
            ctx.drawImage(canvas, 0, 0);
            ctx.filter = 'none';
          }
    
          // 如果启用了自动美颜，应用简单的图像增亮处理
          if (this.autoBeautyEnabled) {
            this.applyProfessionalBeauty(ctx, canvas.width, canvas.height);
          }
    
          // 将canvas转换为图片
          const dataURL = canvas.toDataURL('image/jpeg', 0.95);
    
          // 返回照片数据
          resolve({ 
            path: dataURL, 
            tempImagePath: dataURL,
            timestamp: new Date().toISOString(),
            resolution: `${canvas.width}x${canvas.height}`,
            filter: this.selectedFilter ? this.selectedFilter.id : 'normal',
            beautified: this.autoBeautyEnabled // 标记是否应用了美颜
          });
    
          // 如果启用了美颜，在拍照后提示用户
          if (this.autoBeautyEnabled) {
            // 延迟提示，避免干扰拍照流程
            setTimeout(() => {
              uni.showToast({
                title: '已应用自动美颜',
                icon: 'none',
                duration: 1500
              });
            }, 500);
          }
    
        } catch (e) {
          reject(e);
        }
      });
    },
    confirmPhoto() {
      // 确认当前照片
      this.photos[this.currentPhotoIndex - 1] = {
        local_path: this.tempPhotoPath,
        index: this.currentPhotoIndex,
        filter: this.selectedFilter ? this.selectedFilter.id : 'normal',
        timestamp: new Date().toISOString(),
        resolution: this.videoElement ? 
          `${this.videoElement.videoWidth}x${this.videoElement.videoHeight}` : 
          '1920x1080'
      };
      
      // 判断是否需要继续拍照
      if (this.currentPhotoIndex < this.totalPhotoCount) {
        this.currentPhotoIndex++;
        // 重置状态，回到相机预览
        this.lastPhotoPath = '';
        this.tempPhotoPath = null;
        this.isCapturing = false;
        
        // 修改这里，重置为初始状态
        this.leftCornerIcon = 'back';
        this.leftCornerText = '返回';
        this.rightCornerIcon = 'camera-filled';
        this.rightCornerText = '开始拍摄';
      } else {
        // 所有照片拍摄完成，跳转到预览页面
        this.goToPreview();
      }
    },
    cancelPhoto() {
      // 取消当前照片，重新拍摄
      this.lastPhotoPath = '';
      this.tempPhotoPath = null;
      this.startCapture();
    },
    retakePhoto(index) {
      // 重拍指定位置的照片
      this.currentPhotoIndex = index;
      this.photos[index - 1] = null;
      this.lastPhotoPath = '';
      this.tempPhotoPath = null;
      this.startCapture();
    },
    goToPreview() {
      // 检查是否有照片
      if (!this.hasCompletedPhotos) {
        uni.showToast({
          title: '请至少拍摄一张照片',
          icon: 'none'
        });
        return;
      }
      
      // 过滤掉空值，只保留有效照片
      const validPhotos = this.photos.filter(photo => photo !== null);
      
      // 将照片信息存储到本地
      uni.setStorageSync('captured_photos', validPhotos);
      
      // 跳转到预览页面
      uni.navigateTo({
        url: `/pages/preview/index?template_id=${this.templateId}`
      });
    },
    exitCapture() {
      // 显示确认对话框
      uni.showModal({
        title: '确认取消',
        content: '确定要取消拍照吗？已拍摄的照片将不会保存。',
        confirmText: '确认',
        cancelText: '继续拍照',
        success: (res) => {
          if (res.confirm) {
            // 取消拍照，返回模板选择页面
            uni.navigateBack();
          }
        }
      });
    },
    skipCurrentPhoto() {
      if (this.currentPhotoIndex < this.totalPhotoCount) {
        this.currentPhotoIndex++;
        this.startCapture();
      }
    },
    toggleCameraPosition() {
      this.devicePosition = this.devicePosition === 'front' ? 'back' : 'front';
      
      // 保存当前滤镜
      const currentFilter = this.selectedFilter;
      
      // 如果是Web API，需要重新初始化相机
      if ((this.isElectron || this.isH5) && this.stream) {
        this.stopMediaStream();
        
        // 重新初始化相机
        this.initWebCamera().then(() => {
          // 恢复滤镜
          if (currentFilter && currentFilter.filter && this.videoElement) {
            this.selectFilter(currentFilter);
          }
        });
      }
    },
    toggleFlash() {
      this.flashEnabled = !this.flashEnabled;
    },
    handleCameraError(e) {
      const errMsg = e.detail && e.detail.errMsg ? e.detail.errMsg : '未知错误';
      this.errorMessage = errMsg;
      
      uni.showToast({
        title: '相机启动失败: ' + errMsg,
        icon: 'none',
        duration: 3000
      });
      console.error('相机错误:', e);
      
      // 标记相机不可用
      this.canUseCamera = false;
    },
    retryCamera() {
      this.canUseCamera = true;
      this.errorMessage = '';
      
      // 清理可能存在的资源
      this.stopMediaStream();
      
      // 重新初始化相机
      this.initCamera();
    },
    useTestImage() {
      // 在Electron环境中，可以使用测试图片替代相机拍摄
      const testImageIndex = Math.floor(Math.random() * this.testImages.length);
      const testImagePath = this.testImages[testImageIndex];
      
      // 模拟相机拍照效果
      this.isFlashing = true;
      setTimeout(() => {
        this.isFlashing = false;
        this.lastPhotoPath = testImagePath;
        this.tempPhotoPath = testImagePath;
      }, 300);
      
      uni.showToast({
        title: '已使用测试图片',
        icon: 'none'
      });
    },
    playCountdownVoice(number) {
      // 这里实现语音播放逻辑
      console.log(`播放倒计时语音: ${number}`);
    },
    // 选择滤镜
    selectFilter(filter) {
      this.selectedFilter = filter;
      
      // 如果有视频元素，可以实时预览滤镜效果
      if (this.videoElement) {
        this.videoElement.style.filter = filter.filter;
      }
    },
    handleLeftCornerClick() {
      if (this.lastPhotoPath) {
        // 如果有照片，则取消当前照片
        this.cancelPhoto();
      } else if (this.isCapturing) {
        // 如果正在拍摄，则取消拍摄
        this.cancelCapture();
      } else {
        // 初始状态，返回模板页面
        this.exitCapture();
      }
    },
    
    handleRightCornerClick() {
      if (this.lastPhotoPath) {
        // 如果有照片，则确认当前照片
        this.confirmPhoto();
      } else if (this.isCapturing) {
        // 如果正在拍摄，则确认拍摄
        this.confirmCapture();
      } else {
        // 初始状态，开始拍摄
        this.startCapture();
      }
    },
    
    startCountdown() {
      this.countdown = this.countdownSeconds;
      this.countdownTimer = setInterval(() => {
        this.countdown--;
        
        // 播放语音提示
        this.playCountdownVoice(this.countdown);
        
        if (this.countdown <= 0) {
          clearInterval(this.countdownTimer);
          this.isCounting = false;
          this.takePhoto();
        }
      }, 1000);
    },
    
    cancelCapture() {
      this.isCapturing = false;
      this.countdown = 0;
      this.isCounting = false;
      
      // 修改这里，重置为初始状态
      this.leftCornerIcon = 'back';
      this.leftCornerText = '返回';
      this.rightCornerIcon = 'camera-filled';
      this.rightCornerText = '开始拍摄';
    },
    
    confirmCapture() {
      this.isCapturing = false;
      this.capturePhoto();
    },
    
    updatePreview() {
      // 更新预览缩略图
      this.$forceUpdate();
    },
    
    // 普通美颜
    applySimpleBeauty(ctx, width, height) {
      try {
        // 获取图像数据
        const imageData = ctx.getImageData(0, 0, width, height);
        const data = imageData.data;
        
        // 亮度和对比度参数（可以根据需要调整）
        const brightness = 15; // 亮度增加量 (0-255)
        const contrast = 1.2;  // 对比度倍数 (1.0是原始对比度)
        
        // 计算对比度的因子
        const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));
        
        // 遍历所有像素，应用亮度和对比度调整
        for (let i = 0; i < data.length; i += 4) {
          // 增加亮度
          data[i] = data[i] + brightness;         // R
          data[i + 1] = data[i + 1] + brightness; // G
          data[i + 2] = data[i + 2] + brightness; // B
          
          // 应用对比度
          data[i] = factor * (data[i] - 128) + 128;         // R
          data[i + 1] = factor * (data[i + 1] - 128) + 128; // G
          data[i + 2] = factor * (data[i + 2] - 128) + 128; // B
          
          // 确保RGB值在0-255范围内
          data[i] = Math.max(0, Math.min(255, data[i]));
          data[i + 1] = Math.max(0, Math.min(255, data[i + 1]));
          data[i + 2] = Math.max(0, Math.min(255, data[i + 2]));
        }
        
        // 将处理后的图像数据放回canvas
        ctx.putImageData(imageData, 0, 0);
        
        console.log('已应用美颜处理');
        return true;
      } catch (e) {
        console.error('应用美颜处理失败:', e);
        return false;
      }
    },
    toggleAutoBeauty(e) {
      this.autoBeautyEnabled = e.detail.value;
    },
    // 应用专业美颜（多种美颜效果综合处理）
    applyProfessionalBeauty(ctx, width, height) {
      try {
        // 获取图像数据
        const imageData = ctx.getImageData(0, 0, width, height);
        const data = imageData.data;
        
        // 美颜参数设置（可根据需要调整）
        const params = {
          brightness: 15,    // 亮度增加量 (0-255)
          contrast: 1.2,     // 对比度倍数 (1.0是原始对比度)
          smoothing: 0.6,    // 磨皮强度 (0-1)
          whitening: 0.3,    // 美白强度 (0-1)
          saturation: 1.1    // 饱和度调整 (1.0是原始饱和度)
        };
        
        // 创建临时数组存储处理后的数据
        const tempData = new Uint8ClampedArray(data);
        
        // 计算对比度的因子
        const contrastFactor = (259 * (params.contrast + 255)) / (255 * (259 - params.contrast));
        
        // 第一阶段：磨皮处理（高斯模糊的简化实现）
        // 使用3x3的模糊核进行简单磨皮
        for (let y = 1; y < height - 1; y++) {
          for (let x = 1; x < width - 1; x++) {
            const centerIdx = (y * width + x) * 4;
            
            // 对每个RGB通道分别进行模糊处理
            for (let c = 0; c < 3; c++) {
              // 收集周围像素值
              let sum = 0;
              for (let dy = -1; dy <= 1; dy++) {
                for (let dx = -1; dx <= 1; dx++) {
                  const idx = ((y + dy) * width + (x + dx)) * 4 + c;
                  sum += data[idx];
                }
              }
              // 计算平均值并应用磨皮强度
              const avg = sum / 9;
              tempData[centerIdx + c] = data[centerIdx + c] * (1 - params.smoothing) + avg * params.smoothing;
            }
          }
        }
        
        // 将模糊后的数据拷贝回主数组
        for (let i = 0; i < data.length; i++) {
          data[i] = tempData[i];
        }
        
        // 第二阶段：美白、对比度和亮度调整
        for (let i = 0; i < data.length; i += 4) {
          // 美白处理（提高亮度同时保持自然）
          let r = data[i];
          let g = data[i + 1];
          let b = data[i + 2];
          
          // 计算像素亮度
          const luminance = 0.299 * r + 0.587 * g + 0.114 * b;
          
          // 美白效果（亮度越低的区域提升越多）
          const whiteningFactor = 1 + params.whitening * (1 - luminance / 255);
          r = Math.min(255, r * whiteningFactor);
          g = Math.min(255, g * whiteningFactor);
          b = Math.min(255, b * whiteningFactor);
          
          // 增加亮度
          r += params.brightness;
          g += params.brightness;
          b += params.brightness;
          
          // 应用对比度
          r = contrastFactor * (r - 128) + 128;
          g = contrastFactor * (g - 128) + 128;
          b = contrastFactor * (b - 128) + 128;
          
          // 调整饱和度
          const newLuminance = 0.299 * r + 0.587 * g + 0.114 * b;
          r = newLuminance + params.saturation * (r - newLuminance);
          g = newLuminance + params.saturation * (g - newLuminance);
          b = newLuminance + params.saturation * (b - newLuminance);
          
          // 确保RGB值在0-255范围内
          data[i] = Math.max(0, Math.min(255, r));
          data[i + 1] = Math.max(0, Math.min(255, g));
          data[i + 2] = Math.max(0, Math.min(255, b));
        }
        
        // 将处理后的图像数据放回canvas
        ctx.putImageData(imageData, 0, 0);
        
        console.log('已应用专业美颜处理');
        return true;
      } catch (e) {
        console.error('应用美颜处理失败:', e);
        return false;
      }
    },
  }
}
</script>

<style>
.app-container {
  position: relative;
  width: 100%;
  height: 100vh;
  background-color: #000;
  display: flex;
}

/* 左侧预览栏 */
.preview-sidebar {
  width: 240px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  z-index: 10;
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
}

.preview-item {
  position: relative;
  width: 200px;
  height: 150px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
  background-color: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(5px);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.preview-item image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.preview-item .empty {
  background-color: transparent;
  display: flex;
  justify-content: center;
  align-items: center;
  color: rgba(255, 255, 255, 0.7);
  font-size: 24px;
  height: 100%;
  width: 100%;
}

.preview-number {
  position: absolute;
  top: 8px;
  left: 8px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: 600;
  font-size: 14px;
}

.preview-item.current .preview-number {
  background-color: #1a73e8;
}

.retake-btn {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 16px;
  transition: all 0.2s ease;
}

.retake-btn:active {
  background-color: rgba(0, 0, 0, 0.8);
  transform: scale(1.1);
}

/* 相机视图 */
.camera-view {
  flex: 1;
  position: relative;
  overflow: hidden;
}

.camera-container {
  width: 100%;
  height: 100%;
  position: relative;
  background-color: #000;
}

.camera {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.flash {
  background-color: #fff;
}

.flash-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: white;
  opacity: 0;
  z-index: 30;
  pointer-events: none;
  transition: opacity 0.1s ease-in-out;
}

.flash-active {
  opacity: 1;
}

.captured-photo-display {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 5;
}

.countdown {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  justify-content: center;
  align-items: center;
  width: 150px;
  height: 150px;
  border-radius: 75px;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 10;
}

.countdown-number {
  font-size: 80px;
  font-weight: bold;
  color: white;
}

/* 顶部操作按钮 */
.action-buttons {
  position: absolute;
  top: 30px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  padding: 0 100px;
  z-index: 20;
}

/* 照片操作按钮 */
.photo-action-buttons {
  position: absolute;
  bottom: 50px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  gap: 30px;
  z-index: 20;
}

.action-btn {
  padding: 20px 50px;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
  transition: all 0.2s ease;
}

.cancel-btn {
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
}

.confirm-btn {
  background-color: #34a853;
  color: white;
}

.btn-disabled {
  background-color: rgba(52, 168, 83, 0.5);
  opacity: 0.7;
  pointer-events: none;
}

.web-camera {
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 4;
}

.web-camera-placeholder {
  width: 100%;
  height: 100%;
  background-color: #222;
  display: flex;
  justify-content: center;
  align-items: center;
  color: rgba(255, 255, 255, 0.7);
  font-size: 16px;
}

.camera-error {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 20px;
  text-align: center;
  z-index: 100;
}

.error-icon {
  font-size: 48px;
  margin-bottom: 20px;
}

.error-message {
  font-size: 18px;
  max-width: 80%;
  line-height: 1.5;
  margin-bottom: 20px;
}

.retry-btn {
  background-color: #1a73e8;
  color: white;
  padding: 12px 24px;
}

/* 增加底部按钮区域 */
.bottom-controls {
  position: absolute;
  bottom: 20px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  gap: 20px;
  z-index: 15;
}

.control-btn {
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  width: 50px;
  height: 50px;
  border-radius: 25px;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.2s ease;
}

.control-btn:active {
  background-color: rgba(0, 0, 0, 0.8);
  transform: scale(1.1);
}

/* 调试信息区域 */
.debug-info {
  position: absolute;
  bottom: 10px;
  left: 10px;
  background-color: rgba(0, 0, 0, 0.7);
  color: rgba(255, 255, 255, 0.8);
  font-size: 12px;
  padding: 8px;
  border-radius: 4px;
  max-width: 300px;
  overflow: hidden;
  z-index: 1000;
  font-family: monospace;
}

/* 滤镜选择器样式 */
.filter-selector {
  position: absolute;
  bottom: 100px;
  width: 100%;
  padding: 10px 0;
  display: flex;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(10px);
}

.filter-scroll {
  white-space: nowrap;
  width: 100%;
  padding: 0 20px;
}

.filter-option {
  display: inline-block;
  margin: 0 10px;
  width: 64px;
  text-align: center;
  transition: all 0.3s;
}

.filter-option.active {
  transform: translateY(-5px);
}

.filter-preview {
  width: 64px;
  height: 64px;
  border-radius: 8px;
  background-color: #555;
  margin-bottom: 8px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 2px solid transparent;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  background-image: url('/static/filter_preview.jpg');
  background-size: cover;
  background-position: center;
}

.filter-option.active .filter-preview {
  border-color: #1a73e8;
}

.filter-name {
  font-size: 12px;
  color: white;
  max-width: 64px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.start-capture-btn {
  background-color: #1a73e8;
  width: 60px;
  height: 60px;
  border-radius: 30px;
}

.start-capture-btn:active {
  background-color: #1557b0;
}

/* 添加新的样式 */
.corner-btn {
  position: absolute;
  top: 20px;
  z-index: 100;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 12px;
  transition: all 0.3s ease;
  min-width: 80px;
  min-height: 80px;
}

.corner-btn:active {
  background: rgba(0, 0, 0, 0.7);
  transform: scale(1.05);
}

.left-corner {
  left: 20px;
}

.right-corner {
  right: 20px;
}

.corner-text {
  font-size: 16px;
  color: #ffffff;
  margin-top: 8px;
  font-weight: 500;
}

/* 为图标增加样式 */
.corner-btn .uni-icons {
  font-size: 28px !important;
}

/* 调整其他样式以适应新布局 */
.camera-view {
  position: relative;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.action-buttons {
  display: none; /* 隐藏原有的顶部按钮 */
}

/* 在页面某个位置（比如你想隐藏但保留功能，可以不添加此UI部分） */
.beauty-switch {
  position: absolute;
  bottom: 10px;
  left: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 10px;
  border-radius: 8px;
}

.beauty-switch text {
  color: white;
  font-size: 16px;
  font-weight: 500;
}

.beauty-switch switch {
  color: #1a73e8;
}
</style> 