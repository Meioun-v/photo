<template>
  <view class="app-container" :style="backgroundStyle">
    <view class="preview-container">
      <!-- 主预览区域 -->
      <view class="preview-main">
        <view class="template-preview" id="templatePreview">
          <!-- 模板合成效果图 -->
          <image v-if="hasPhotos" :src="currentPhoto.local_path" mode="aspectFit" class="preview-image"></image>
          <view v-else class="no-photo-placeholder">
            <uni-icons type="image" size="50" color="#cccccc"></uni-icons>
            <text class="placeholder-text">没有找到照片</text>
          </view>
        </view>
      </view>
      
      <!-- 右侧功能栏 -->
      <view class="action-sidebar">
        <button class="action-btn btn-complete" id="completeBtn" @click="goHome">
          <view class="btn-icon">
            <uni-icons type="checkmarkempty" size="24" color="#ffffff"></uni-icons>
          </view>
          <text>完成</text>
        </button>
        
        <button class="action-btn btn-qrcode" id="qrcodeBtn" @click="handleShare">
          <view class="btn-icon">
            <uni-icons type="link" size="24" color="#ffffff"></uni-icons>
          </view>
          <text>二维码</text>
        </button>
        
        <button class="action-btn btn-print" id="printBtn" @click="handlePrint">
          <view class="btn-icon">
            <uni-icons type="paperplane" size="24" color="#ffffff"></uni-icons>
          </view>
          <text>打印</text>
        </button>
        
        <button class="action-btn btn-gif" id="gifBtn" @click="handleGifCreation">
          <view class="btn-icon">
            <text class="icon-text">GIF</text>
          </view>
          <text>创建GIF</text>
        </button>
        
        <!-- 照片信息 -->
        <view class="photo-info">
          <view class="info-title">照片信息</view>
          
          <view class="info-item">
            <text class="info-label">拍摄时间：</text>
            <text class="info-value" id="photoTime">{{photoTime}}</text>
          </view>
          
          <view class="info-item">
            <text class="info-label">照片尺寸：</text>
            <text class="info-value">{{photoSize}}</text>
          </view>
          
          <view class="info-item">
            <text class="info-label">模板名称：</text>
            <text class="info-value">{{templateName}}</text>
          </view>
        </view>
      </view>
    </view>
    
    <!-- 二维码模态框 -->
    <uni-popup ref="qrcodePopup" type="center" background-color="rgba(0,0,0,0.5)">
      <view class="modal-container">
        <view class="modal-header">
          <view class="modal-title">扫描二维码</view>
          <view class="modal-close" @click="closeQrcodePopup">
            <uni-icons type="close" size="20" color="#333333"></uni-icons>
          </view>
        </view>
        <view class="modal-body">
          <view class="qrcode-container">
            <image :src="qrcodeLocalPath || '/static/qrcode-placeholder.png'" 
                   alt="二维码" 
                   class="qrcode-image"
                   mode="aspectFit"></image>
          </view>
          <view class="text-center text-gray-600 mb-2">扫描二维码获取照片</view>
          <view class="text-center text-gray-500">照片将保存24小时</view>
        </view>
        <view class="modal-footer">
          <button class="btn btn-outline" @click="closeQrcodePopup">关闭</button>
          <button class="btn btn-primary" @click="saveQrcode" :disabled="!qrcodeLocalPath">保存二维码</button>
        </view>
      </view>
    </uni-popup>
    
    <!-- 打印模态框 -->
    <!--
    <uni-popup ref="printPopup" type="center" background-color="rgba(0,0,0,0.5)">
      <view class="modal-container">
        <view class="modal-header">
          <view class="modal-title">打印预览</view>
          <view class="modal-close" @click="closePrintPopup">
            <uni-icons type="close" size="20" color="#333333"></uni-icons>
          </view>
        </view>
        <view class="modal-body">
          <view class="print-preview-container">
            <image :src="currentPhoto.local_path" alt="打印预览" class="print-preview-image"></image>
          </view>
          
          <view class="print-count-container">
            <text class="print-count-label">打印份数：</text>
            <view class="count-btn decrease-btn" @click="decreasePrintCount">
              <uni-icons type="minus" size="16" color="#333333"></uni-icons>
            </view>
            <text class="print-count-value">{{printCount}}</text>
            <view class="count-btn increase-btn" @click="increasePrintCount">
              <uni-icons type="plus" size="16" color="#333333"></uni-icons>
            </view>
          </view>
        </view>
        <view class="modal-footer">
          <button class="btn btn-outline" @click="closePrintPopup">取消</button>
          <button class="btn btn-primary" @click="confirmPrint">确认打印</button>
        </view>
      </view>
    </uni-popup>
    -->
    
    <!-- 打印进度模态框 -->
    <!--
    <uni-popup ref="printProgressPopup" type="center" background-color="rgba(0,0,0,0.5)" :mask-click="false">
      <view class="modal-container">
        <view class="modal-header">
          <view class="modal-title">打印中</view>
        </view>
        <view class="modal-body">
          <view class="print-icon-container">
            <uni-icons type="paperplane" size="60" color="#1a73e8" class="print-icon"></uni-icons>
          </view>
          <view class="text-center text-gray-600 mb-4">正在打印照片，请稍候...</view>
          <view class="progress-bar-container">
            <view class="progress-bar" :style="{width: printProgress + '%'}"></view>
          </view>
          <view class="text-center text-gray-500">{{printStatus}}</view>
        </view>
      </view>
    </uni-popup>
    -->
    
    <!-- GIF模态框 -->
    <uni-popup ref="gifPopup" type="center" background-color="rgba(0,0,0,0.5)">
      <view class="modal-container">
        <view class="modal-header">
          <view class="modal-title">GIF动画预览</view>
          <view class="modal-close" @click="closeGifPopup">
            <uni-icons type="close" size="20" color="#333333"></uni-icons>
          </view>
        </view>
        <view class="modal-body">
          <view class="gif-preview-container">
            <view v-if="creatingGif" class="gif-loading">
              <image src="/static/gif-loading.gif" class="loading-spinner"></image>
              <text class="loading-text">正在生成GIF动画，请稍候...</text>
            </view>
            <image v-else-if="gifUrl" :src="gifUrl" mode="aspectFit" class="gif-preview-image"></image>
            <view v-else class="gif-placeholder">
              <text class="gif-placeholder-text">点击"生成GIF"按钮创建动画</text>
            </view>
          </view>
          
          <view class="gif-options" v-if="!creatingGif">
            <view class="option-item">
              <text class="option-label">动画速度：</text>
              <view class="speed-slider">
                <text class="speed-label">慢</text>
                <slider 
                  class="speed-control" 
                  min="100" 
                  max="2000" 
                  step="100" 
                  :value="gifDelay" 
                  @change="onGifDelayChange" 
                  show-value
                  activeColor="#1a73e8"
                  backgroundColor="#e0e0e0"
                ></slider>
                <text class="speed-label">快</text>
              </view>
            </view>
          </view>
        </view>
        <view class="modal-footer">
          <button class="btn btn-outline" @click="closeGifPopup">关闭</button>
          <button class="btn btn-primary" @click="createGif" :disabled="creatingGif">
            {{ gifUrl ? '重新生成' : '生成GIF' }}
          </button>
          <button class="btn btn-success" @click="saveGif" :disabled="!gifUrl || creatingGif">保存GIF</button>
        </view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'
import uniPopup from '@/uni_modules/uni-popup/components/uni-popup/uni-popup.vue'
import { getAppBackground } from '@/uni_modules/uni-id-pages/pages/login/login-withpwd.vue'
// 使用我们自己的helper模块
import { createGIF, loadImage, saveBlob } from '@/utils/gifHelper.js'
import printService from '@/utils/printService.js'

export default {
  components: {
    uniIcons,
    uniPopup
  },
  data() {
    return {
      templateId: '',
      photos: [],
      currentIndex: 0,
      photoTime: '',
      templateName: '经典模板1',
      printCount: 1,
      printProgress: 0,
      printStatus: '正在打印第1张，共1张',
      printInterval: null,
      backgroundImage: '/static/login-bg.jpg', // 默认背景
      photoSize: '2400 x 3600',
      templateInfo: null, // 存储模板信息
      compositePhotos: null, // 合成照片结果
      loadingError: null, // 加载错误信息
      taskId: '', // 添加taskId字段
      uploadedFiles: {  // 添加上传文件记录
        originals: [],
        composite: '',
        gif: ''  // 添加 GIF 字段
      },
      qrcodeUrl: '', // 新增：二维码云存储URL
      qrcodeLocalPath: '', // 新增：二维码本地路径
      
      // GIF 相关数据
      gifUrl: '', // GIF URL
      creatingGif: false, // 是否正在创建GIF
      gifDelay: 500, // GIF 帧延迟（毫秒）
      gifBlob: null, // GIF Blob 对象
    }
  },
  computed: {
    currentPhoto() {
      // 首先检查是否有合成照片
      if (this.compositePhotos) {
        return this.compositePhotos;
      }
      
      // 如果没有合成照片，则返回第一张原始照片
      return this.photos[this.currentIndex] || {};
    },
    hasPhotos() {
      return this.photos && this.photos.length > 0;
    },
    backgroundStyle() {
      if (this.backgroundImage.startsWith('#')) {
        return {
          backgroundColor: this.backgroundImage
        };
      }
      
      return {
        backgroundImage: `url('${this.backgroundImage}')`,
        backgroundSize: 'cover',
        backgroundPosition: 'center',
        backgroundRepeat: 'no-repeat',
        backgroundAttachment: 'fixed'
      };
    }
  },
  async onLoad(options) {
    this.templateId = options.template_id || '';
    console.log('预览页面加载，模板ID:', this.templateId);
    
    try {
      // 加载背景图
      this.backgroundImage = await getAppBackground();
      console.log('成功获取背景图:', this.backgroundImage);
    } catch (e) {
      console.error('获取背景图失败:', e);
    }
    
    this.loadPhotos();
    this.setPhotoTime();
    await this.loadTemplateInfo();
    
    // 设置照片尺寸
    if (this.templateInfo) {
      this.photoSize = `${this.templateInfo.width || 2400} x ${this.templateInfo.height || 3600}`;
    }
    
    // 更新模板名称
    if (this.templateInfo) {
      this.templateName = this.templateInfo.name || '经典模板1';
    } else {
      this.setTemplateName();
    }
    
    // 尝试合成照片(GIF生成和上传都在createCompositePhoto方法中处理)
    await this.createCompositePhoto();
  },
  methods: {
    loadPhotos() {
      console.log('加载照片...');
      // 从本地存储获取照片
      const capturedPhotos = uni.getStorageSync('captured_photos');
      if (capturedPhotos && capturedPhotos.length > 0) {
        // 确保照片对象格式正确
        this.photos = capturedPhotos.map(photo => {
          if (typeof photo === 'string') {
            return { local_path: photo };
          }
          return photo;
        });
        console.log('成功加载照片:', this.photos.length, '张', this.photos);
      } else {
        console.warn('没有找到照片');
        uni.showToast({
          title: '没有找到照片',
          icon: 'none'
        });
      }
    },
    setPhotoTime() {
      const now = new Date();
      const year = now.getFullYear();
      const month = String(now.getMonth() + 1).padStart(2, '0');
      const day = String(now.getDate()).padStart(2, '0');
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      this.photoTime = `${year}-${month}-${day} ${hours}:${minutes}`;
    },
    // 加载模板信息
    async loadTemplateInfo() {
      if (!this.templateId) {
        console.warn('模板ID为空，无法加载模板信息');
        return;
      }
      
      try {
        console.log('正在加载模板信息...');
        const db = uniCloud.database();
        const templatesCollection = db.collection('a-templates');
        
        const { result } = await templatesCollection.doc(this.templateId).get();
        
        if (result && result.data) {
          // 确保我们获取到的是单个模板对象
          this.templateInfo = Array.isArray(result.data) ? result.data[0] : result.data;
          console.log('成功加载模板信息:', this.templateInfo);
          
          // 更新照片尺寸显示
          if (this.templateInfo) {
            this.photoSize = `${this.templateInfo.width || 1920} x ${this.templateInfo.height || 2880}`;
          }
        } else {
          console.warn('未找到模板信息');
        }
      } catch (e) {
        console.error('加载模板信息失败:', e);
      }
    },
    // 计算适合插槽的图片尺寸（保持比例）
    calculateImageDimensions(imgWidth, imgHeight, slotWidth, slotHeight) {
      const imgRatio = imgWidth / imgHeight;
      const slotRatio = slotWidth / slotHeight;
      
      // 始终使用插槽的宽度和高度
      return {
        width: slotWidth,
        height: slotHeight
      };
    },
    
    // 创建合成照片
    async createCompositePhoto() {
      if (!this.hasPhotos) {
        console.warn('没有照片可供合成');
        return;
      }
      
      try {
        console.log('开始照片合成...');
        console.log('当前模板信息:', this.templateInfo);
        console.log('当前照片:', this.photos);
        
        // 检查模板信息是否存在
        if (!this.templateInfo) {
          console.warn('模板信息不存在，使用第一张照片作为合成结果');
          this.compositePhotos = this.photos[0];
          return;
        }
        
        // 创建离屏Canvas
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        
        // 设置画布尺寸为模板尺寸
        canvas.width = this.templateInfo.width;
        canvas.height = this.templateInfo.height;
        
        console.log(`创建画布，尺寸: ${canvas.width} x ${canvas.height}`);
        
        // 绘制背景色
        ctx.fillStyle = '#FFFFFF';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        // 如果有背景图，先绘制背景图
        if (this.templateInfo.background_images && this.templateInfo.background_images.length > 0) {
          console.log('准备绘制背景图...');
          const bgImage = this.templateInfo.background_images[0];
          
          try {
            // 加载背景图
            console.log('加载背景图:', bgImage.image_url);
            const backgroundImg = await this.loadImage(bgImage.image_url);
            ctx.drawImage(
              backgroundImg, 
              bgImage.position_x || 0, 
              bgImage.position_y || 0,
              bgImage.width || canvas.width,
              bgImage.height || canvas.height
            );
            console.log('背景图绘制完成');
          } catch (e) {
            console.error('背景图加载失败:', e);
          }
        }
        
        // 处理照片插槽
        if (this.templateInfo.slots && this.templateInfo.slots.length > 0) {
          console.log(`开始绘制${this.templateInfo.slots.length}个照片插槽...`);
          
          // 按照 photo_index 分组处理插槽
          const slotsByPhotoIndex = new Map();
          
          // 将插槽按照 photo_index 分组
          this.templateInfo.slots.forEach(slot => {
            if (!slotsByPhotoIndex.has(slot.photo_index)) {
              slotsByPhotoIndex.set(slot.photo_index, []);
            }
            slotsByPhotoIndex.get(slot.photo_index).push(slot);
          });
          
          // 遍历每个 photo_index 及其对应的插槽
          for (const [photoIndex, slots] of slotsByPhotoIndex) {
            // 获取对应的照片（索引从1开始，所以要减1）
            const photo = this.photos[photoIndex - 1];
            
            if (!photo || !photo.local_path) {
              console.warn(`照片 ${photoIndex} 不存在或路径无效`);
              continue;
            }
            
            try {
              // 只需要加载一次照片
              console.log(`加载照片 ${photoIndex}, 路径: ${photo.local_path}`);
              const img = await this.loadImage(photo.local_path);
              
              // 将这张照片绘制到所有对应的插槽中
              for (const slot of slots) {
                console.log(`绘制照片 ${photoIndex} 到插槽:`, slot);
                
                // 直接使用插槽定义的位置和尺寸
                ctx.drawImage(
                  img,
                  slot.position_x,
                  slot.position_y,
                  slot.width,
                  slot.height
                );
                
                console.log(`照片 ${photoIndex} 在插槽位置(${slot.position_x}, ${slot.position_y})绘制完成`);
              }
            } catch (e) {
              console.error(`照片 ${photoIndex} 加载或绘制失败:`, e);
            }
          }
        }
        
        // 添加边框或其他装饰效果（可选）
        this.drawBorder(ctx, canvas.width, canvas.height);
        
        // 将Canvas转换为图片
        const compositeImageUrl = canvas.toDataURL('image/jpeg', 0.9);
        console.log('合成照片完成');
        
        // 保存合成照片
        this.compositePhotos = {
          local_path: compositeImageUrl,
          composite: true
        };
        
        // 保存合成照片到本地存储
        uni.setStorageSync('composite_photo', compositeImageUrl);
        
        // 上传所有资源
        await this.uploadPhotos();
        
      } catch (e) {
        console.error('照片合成失败:', e);
        this.loadingError = '照片合成失败：' + e.message;
        
        // 使用第一张照片作为备选
        if (this.photos.length > 0) {
          this.compositePhotos = this.photos[0];
        }
      }
    },
    
    // 加载图片Promise
    loadImage(src) {
      return new Promise((resolve, reject) => {
        console.log('开始加载图片:', src);
        
        const img = new Image();
        img.crossOrigin = 'anonymous'; // 允许跨域
        
        img.onload = () => {
          console.log('图片加载成功:', src);
          resolve(img);
        };
        
        img.onerror = (e) => {
          console.error('图片加载失败:', src, e);
          reject(new Error('图片加载失败'));
        };
        
        // 处理各种类型的图片路径
        if (src.startsWith('data:')) {
          img.src = src;
        } else if (src.startsWith('http') || src.startsWith('https')) {
          // 对于远程图片，添加时间戳避免缓存
          img.src = src + (src.includes('?') ? '&' : '?') + '_t=' + new Date().getTime();
        } else if (src.startsWith('cloud://')) {
          // 云存储路径暂时使用默认图片
          img.src = '/static/placeholder.jpg';
          console.warn('云存储图片暂不支持，使用默认图片代替:', src);
        } else {
          // 本地路径
          if (window && window.process && window.process.type === 'renderer') {
            img.src = `file://${src}`;
          } else {
            img.src = src;
          }
        }
      });
    },
    
    // 绘制边框
    drawBorder(ctx, width, height) {
      // 简单边框
      ctx.strokeStyle = '#FFFFFF';
      ctx.lineWidth = 20;
      ctx.strokeRect(10, 10, width - 20, height - 20);
      
      ctx.strokeStyle = '#000000';
      ctx.lineWidth = 2;
      ctx.strokeRect(20, 20, width - 40, height - 40);
    },
    
    // 导出合成照片
    exportCompositePhoto() {
      if (!this.compositePhotos || !this.compositePhotos.local_path) {
        uni.showToast({
          title: '没有可导出的合成照片',
          icon: 'none'
        });
        return;
      }
      
      try {
        // 将Base64图片保存到本地（在Electron环境中需要特殊处理）
        if (this.compositePhotos.local_path.startsWith('data:')) {
          if (window && window.process && window.process.type === 'renderer') {
            // Electron环境
            console.log('Electron环境下保存图片');
            // 注意：这里需要Electron特定的API实现，暂用示例代码
            uni.showToast({
              title: '照片已导出到桌面',
              icon: 'success'
            });
          } else {
            // 普通浏览器环境
            const a = document.createElement('a');
            a.href = this.compositePhotos.local_path;
            a.download = `photobooth_${new Date().getTime()}.jpg`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            
            uni.showToast({
              title: '照片已下载',
              icon: 'success'
            });
          }
        } else {
          // 其他类型的路径（如本地文件系统路径）
          uni.saveImageToPhotosAlbum({
            filePath: this.compositePhotos.local_path,
            success: () => {
              uni.showToast({
                title: '照片已保存到相册',
                icon: 'success'
              });
            },
            fail: (err) => {
              console.error('保存照片失败:', err);
              uni.showToast({
                title: '保存照片失败',
                icon: 'none'
              });
            }
          });
        }
      } catch (e) {
        console.error('导出照片失败:', e);
        uni.showToast({
          title: '导出照片失败',
          icon: 'none'
        });
      }
    },
    setTemplateName() {
      if (this.templateId === 'horizontal-1') {
        this.templateName = '经典横版模板';
      } else if (this.templateId === 'horizontal-2') {
        this.templateName = '多格横版模板';
      } else if (this.templateId === 'vertical-3') {
        this.templateName = '竖版组合模板';
      } else {
        this.templateName = '经典模板1';
      }
    },
    goHome() {
      uni.navigateTo({
        url: '/pages/start/index'
      });
    },
    async handleShare() {
      try {
        // 如果还没有生成二维码，先生成
        if (!this.qrcodeUrl && this.compositePhotos && this.compositePhotos.image_url) {
          await this.updateQRCode(this.compositePhotos.image_url);
        }
        
        // 打开二维码弹窗
        this.$refs.qrcodePopup.open();
      } catch (e) {
        console.error('分享操作失败:', e);
        uni.showToast({
          title: '分享失败',
          icon: 'none'
        });
      }
    },
    closeQrcodePopup() {
      this.$refs.qrcodePopup.close();
    },
    async saveQrcode() {
      try {
        if (!this.qrcodeLocalPath) {
          throw new Error('二维码未生成');
        }
        
        await uni.saveImageToPhotosAlbum({
          filePath: this.qrcodeLocalPath
        });
        
        uni.showToast({
          title: '二维码已保存到相册',
          icon: 'success'
        });
        this.closeQrcodePopup();
      } catch (e) {
        console.error('保存二维码失败:', e);
        uni.showToast({
          title: '保存失败',
          icon: 'none'
        });
      }
    },
    handlePrint() {
      if (!this.compositePhotos || !this.compositePhotos.local_path) {
        uni.showToast({
          title: '没有可打印的照片',
          icon: 'none'
        });
        return;
      }
      
      // 不再打开弹窗，直接执行打印
      // this.$refs.printPopup.open();
      this.directPrint();
    },
    // 修改直接打印方法
    async directPrint() {
      try {
        // 不再显示打印进度弹窗
        // this.$refs.printProgressPopup.open();
        // this.printStatus = `正在打印第1张，共1张`; // 默认打印1份
        // this.printProgress = 0;
        
        // 获取当前照片的本地路径
        const imagePath = this.currentPhoto.local_path;
        if (!imagePath) {
          throw new Error('没有可打印的图片');
        }

        // 不再需要进度条更新
        // this.printInterval = setInterval(() => {
        //   if (this.printProgress < 90) {
        //     this.printProgress += 5;
        //   }
        // }, 200);

        // 显示简单的提示
        uni.showLoading({
          title: '正在打印...',
          mask: true
        });

        // 执行打印 - 只打印1份
        await printService.printMultipleCopies(imagePath, 1);
        
        // 不再需要清除定时器
        // clearInterval(this.printInterval);
        // this.printProgress = 100;
        
        // 更新打印状态
        await this.updatePrintStatus();
        
        // 不再关闭进度弹窗，直接显示成功提示
        uni.hideLoading();
        uni.showToast({
          title: '打印完成！',
          icon: 'success'
        });
        
      } catch (error) {
        // 处理错误
        // clearInterval(this.printInterval);
        // this.$refs.printProgressPopup.close();
        uni.hideLoading();
        
        uni.showToast({
          title: error.message || '打印失败',
          icon: 'none'
        });
      }
    },
    // GIF 相关方法
    handleGifCreation() {
      this.gifUrl = '';
      this.gifBlob = null;
      this.$refs.gifPopup.open();
    },
    
    closeGifPopup() {
      this.$refs.gifPopup.close();
    },
    
    onGifDelayChange(e) {
      // 反转滑块值，因为滑块值越大延迟越小（速度越快）
      this.gifDelay = 2100 - e.detail.value;
    },
    
    async createGif() {
      if (this.creatingGif) return;
      if (!this.photos || this.photos.length === 0) {
        uni.showToast({
          title: '没有找到照片',
          icon: 'none'
        });
        return;
      }
      
      this.creatingGif = true;
      this.gifUrl = '';
      this.gifBlob = null;
      
      try {
        // 加载 GIF.js 库
        console.log('确保 GIF.js 已加载...');
        await this.ensureGifJsLoaded();
        
        // 收集所有照片的路径
        const imagePaths = this.photos.map(photo => photo.local_path);
        
        console.log('创建 GIF 实例...');
        // 直接使用全局 GIF 构造函数
        const gif = new window.GIF({
          workers: 2,
          quality: 10,
          workerScript: '/static/gif.worker.js'
        });
        
        // 获取图片尺寸
        const firstImg = await loadImage(imagePaths[0]);
        const aspectRatio = firstImg.width / firstImg.height;
        
        // 设置 GIF 尺寸
        const gifWidth = 400;
        const gifHeight = Math.round(gifWidth / aspectRatio);
        gif.width = gifWidth;
        gif.height = gifHeight;
        
        // 创建 canvas
        const canvas = document.createElement('canvas');
        canvas.width = gifWidth;
        canvas.height = gifHeight;
        const ctx = canvas.getContext('2d');
        
        // 处理并添加每张图片
        for (const path of imagePaths) {
          const img = await loadImage(path);
          
          // 清空 canvas
          ctx.fillStyle = '#FFFFFF';
          ctx.fillRect(0, 0, gifWidth, gifHeight);
          
          // 绘制图片
          ctx.drawImage(img, 0, 0, gifWidth, gifHeight);
          
          // 添加到 GIF
          gif.addFrame(canvas, { delay: this.gifDelay, copy: true });
        }
        
        // 生成 GIF
        gif.on('finished', (blob) => {
          this.gifBlob = blob;
          this.gifUrl = URL.createObjectURL(blob);
          this.creatingGif = false;
        });
        
        gif.on('progress', (progress) => {
          console.log(`GIF 生成进度: ${Math.round(progress * 100)}%`);
        });
        
        gif.on('error', (error) => {
          console.error('GIF 生成出错:', error);
          uni.showToast({
            title: 'GIF 生成失败',
            icon: 'none'
          });
          this.creatingGif = false;
        });
        
        gif.render();
        
      } catch (error) {
        console.error('创建 GIF 失败:', error);
        uni.showToast({
          title: 'GIF 创建失败: ' + error.message,
          icon: 'none'
        });
        this.creatingGif = false;
      }
    },
    
    // 添加辅助方法确保 GIF.js 已加载
    ensureGifJsLoaded() {
      return new Promise((resolve, reject) => {
        // 如果 GIF 已加载，直接返回
        if (typeof window.GIF === 'function') {
          console.log('GIF.js 已加载');
          return resolve();
        }
        
        console.log('加载 GIF.js 库...');
        // 检查是否已经有脚本标签
        const existingScript = document.querySelector('script[src="/static/gif.js"]');
        
        if (existingScript) {
          // 脚本已添加但可能还未加载完成，等待
          const checkInterval = setInterval(() => {
            if (typeof window.GIF === 'function') {
              clearInterval(checkInterval);
              console.log('GIF.js 加载完成');
              resolve();
            }
          }, 100);
          
          // 设置超时
          setTimeout(() => {
            clearInterval(checkInterval);
            reject(new Error('GIF.js 加载超时'));
          }, 5000);
        } else {
          // 添加脚本
          const script = document.createElement('script');
          script.src = '/static/gif.js';
          
          script.onload = () => {
            console.log('GIF.js 脚本加载完成');
            if (typeof window.GIF === 'function') {
              resolve();
            } else {
              reject(new Error('GIF.js 加载后构造函数不可用'));
            }
          };
          
          script.onerror = () => {
            reject(new Error('GIF.js 脚本加载失败'));
          };
          
          document.head.appendChild(script);
        }
      });
    },
    
    async saveGif() {
      if (!this.gifUrl || !this.gifBlob) {
        uni.showToast({
          title: '请先生成 GIF',
          icon: 'none'
        });
        return;
      }
      
      try {
        // 使用我们的helper函数保存GIF
        const fileName = `photo_booth_${new Date().getTime()}.gif`;
        saveBlob(this.gifBlob, fileName);
        
        // 成功消息
        uni.showToast({
          title: 'GIF 已保存',
          icon: 'success'
        });
        
        // 如果需要，尝试上传到云存储
        if (this.taskId) {
          try {
            const cloudPath = `gifs/${this.taskId}_${new Date().getTime()}.gif`;
            const fileID = await this.uploadToCloud(this.gifUrl, cloudPath);
            this.uploadedFiles.gif = fileID;
            
            // 更新任务记录
            await this.updateTaskRecord({
          gif_url: fileID,  // 添加 GIF URL 到数据库
 // 更新传文件记录
        });
          } catch (uploadError) {
            console.error('上传 GIF 到云存储失败:', uploadError);
            // 不要中断流程，继续执行
          }
        }
        
      } catch (error) {
        console.error('保存 GIF 失败:', error);
        uni.showToast({
          title: '保存 GIF 失败',
          icon: 'none'
        });
      }
    },
    // 上传图片到云存储
    async uploadToCloud(filePath, cloudPath) {
      try {
        console.log('开始上传文件到云存储:', cloudPath);
        const result = await uniCloud.uploadFile({
          filePath: filePath,
          cloudPath: cloudPath
        });
        console.log('文件上传成功:', result);
        // 检查返回数据结构
        if (result && result.fileID) {
          return result.fileID;
        } else if (result && result.result && result.result.fileID) {
          return result.result.fileID;
        } else {
          throw new Error('上传文件失败：没有返回fileID');
        }
      } catch (e) {
        console.error('文件上传失败:', e);
        throw e;
      }
    },
    // 生成二维码图片
    async generateQRCode(url) {
      try {
        console.log('开始生成二维码，URL:', url);
        // 使用在线API生成二维码
        const qrcodeApiUrl = `https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=${encodeURIComponent(url)}`;
        
        // 下载二维码图片到本地临时文件
        return new Promise((resolve, reject) => {
          uni.downloadFile({
            url: qrcodeApiUrl,
            success: (res) => {
              console.log('二维码下载成功:', res);
              if (res.statusCode === 200) {
                resolve(res.tempFilePath);
              } else {
                reject(new Error('二维码下载失败'));
              }
            },
            fail: (err) => {
              console.error('二维码下载失败:', err);
              reject(err);
            }
          });
        });
      } catch (e) {
        console.error('生成二维码失败:', e);
        throw e;
      }
    },
    
    // 更新二维码显示
    async updateQRCode(imageUrl) {
      try {
        console.log('开始更新二维码，图片URL:', imageUrl);
        // 直接使用图片的云存储链接作为二维码内容
        const qrcodePath = await this.generateQRCode(imageUrl);
        this.qrcodeLocalPath = qrcodePath;
        
        // 上传二维码到云存储
        const timestamp = Date.now();
        const cloudPath = `photos/${timestamp}_qrcode.jpg`;
        const qrcodeCloudUrl = await this.uploadToCloud(qrcodePath, cloudPath);
        this.qrcodeUrl = qrcodeCloudUrl;
        
        // 更新任务记录中的result_images
        if (this.taskId) {
          const db = uniCloud.database();
          const taskDoc = await db.collection('a-tasks').doc(this.taskId).get();
          
          if (taskDoc.result && taskDoc.result.data) {
            const taskData = taskDoc.result.data;
            const resultImages = taskData.result_images || [];
            
            // 更新对应图片的qrcode_url
            const updatedResultImages = resultImages.map(img => {
              if (img.image_url === imageUrl) {
                return {
                  ...img,
                  qrcode_url: qrcodeCloudUrl
                };
              }
              return img;
            });
            
            // 更新任务记录
            await this.updateTaskRecord({
              result_images: updatedResultImages
            });
            
            console.log('已更新result_images中的二维码URL:', qrcodeCloudUrl);
          }
        }
        
        return qrcodeCloudUrl;
      } catch (e) {
        console.error('更新二维码失败:', e);
        uni.showToast({
          title: '二维码生成失败',
          icon: 'none'
        });
        throw e;
      }
    },
    // 修改 uploadPhotos 方法
    async uploadPhotos() {
      try {
        console.log('开始上传照片...');
        uni.showLoading({
          title: '正在上传...',
          mask: true
        });
        
        // 准备所有上传任务
        const uploadTasks = [];
        const photos = [];
        const resultImages = [];
        
        // 并行上传原图和缩略图
        for (let i = 0; i < this.photos.length; i++) {
          const photo = this.photos[i];
          const timestamp = Date.now() + i; // 加上索引避免时间戳重复
          
          // 创建原图上传任务
          const originalTask = (async () => {
            const originalPath = `photos/${timestamp}_${i}_original.jpg`;
            const photoUrl = await this.uploadToCloud(photo.local_path, originalPath);
            return { photoUrl, index: i };
          })();
          
          // 创建缩略图上传任务
          const thumbnailTask = (async () => {
            const compressedImage = await this.compressImage(photo.local_path);
            const thumbnailPath = `photos/${timestamp}_${i}_thumb.jpg`;
            const thumbnailUrl = await this.uploadToCloud(compressedImage, thumbnailPath);
            return { thumbnailUrl, index: i };
          })();
          
          uploadTasks.push(originalTask, thumbnailTask);
        }
        
        // 等待所有上传任务完成
        const results = await Promise.all(uploadTasks);
        
        // 处理上传结果
        for (let i = 0; i < this.photos.length; i++) {
          const originalResult = results.find(r => r.photoUrl && r.index === i);
          const thumbnailResult = results.find(r => r.thumbnailUrl && r.index === i);
          
          if (originalResult && thumbnailResult) {
            photos.push({
              photo_url: originalResult.photoUrl,
              thumbnail_url: thumbnailResult.thumbnailUrl,
              index: i + 1
            });
          }
        }
        
        // 并行上传合成图及其相关资源
        if (this.compositePhotos && this.compositePhotos.local_path) {
          const timestamp = Date.now();
          const compositeTasks = [];
          
          // 上传合成图任务
          const compositeTask = (async () => {
            const compositePath = `photos/${timestamp}_composite.jpg`;
            return await this.uploadToCloud(this.compositePhotos.local_path, compositePath);
          })();
          
          // 上传合成图缩略图任务
          const compositeThumbTask = (async () => {
            const compressedComposite = await this.compressImage(this.compositePhotos.local_path);
            const thumbnailPath = `photos/${timestamp}_composite_thumb.jpg`;
            return await this.uploadToCloud(compressedComposite, thumbnailPath);
          })();
          
          compositeTasks.push(compositeTask, compositeThumbTask);
          
          // 等待合成图相关上传完成
          const [compositeImageUrl, thumbnailUrl] = await Promise.all(compositeTasks);
          
          // 生成分享二维码
          const qrcodePath = await this.updateQRCode(compositeImageUrl);
          
          resultImages.push({
            image_url: compositeImageUrl,
            thumbnail_url: thumbnailUrl,
            qrcode_url: qrcodePath,
            template_id: this.templateId
          });
        }
        
        // 创建任务记录
        await this.createTaskWithPhotos(photos, resultImages);
        
        uni.hideLoading();
        uni.showToast({
          title: '上传完成',
          icon: 'success'
        });
        
      } catch (e) {
        console.error('照片上传失败:', e);
        uni.hideLoading();
        uni.showToast({
          title: '照片上传失败',
          icon: 'none'
        });
      }
    },
    // 修改createTaskWithPhotos方法，添加gif_url字段
    async createTaskWithPhotos(photos, resultImages) {
      console.log('创建带照片的任务记录...');
      
      try {
        // 检查是否已经有taskId
        if (this.taskId) {
          console.log('任务记录已存在，仅更新照片信息');
          await this.updateTaskRecord({
            photos: photos,
            result_images: resultImages,
            status: 1, // 1-已完成
            complete_date: Date.now()
          });
          return;
        }

        // 检查登录状态
        const auth = uniCloud.getCurrentUserInfo();
        console.log('当前用户信息:', auth);
        
        if (!auth.uid) {
          console.log('用户未登录，跳转到登录页面');
          uni.showToast({
            title: '请先登录',
            icon: 'none'
          });
          
          // 保存当前页面路径
          const currentPage = '/pages/preview/index';
          const params = `?template_id=${this.templateId}`;
          uni.setStorageSync('redirect_page', currentPage + params);
          
          // 跳转到登录页
          setTimeout(() => {
            uni.redirectTo({
              url: '/uni_modules/uni-id-pages/pages/login/login-withpwd'
            });
          }, 1500);
          return;
        }
        
        // 从模板中获取需要拍摄的照片数量
        const photoCount = this.templateInfo?.photo_count || this.photos.length;
        
        // 创建完整的任务数据，包含照片信息
        const taskData = {
          operator_id: auth.uid,
          template_id: this.templateId,
          photo_count: photoCount,
          photos: photos,
          result_images: resultImages,
          status: 1,
          print_status: 0,
          create_date: Date.now(),
          complete_date: Date.now()
        };
        
        console.log('准备创建带照片的任务记录:', taskData);
        const db = uniCloud.database();
        const result = await db.collection('a-tasks').add(taskData);
        console.log('创建任务结果:', result);
        
        // 检查返回结果中的ID
        if (result && result.result && result.result.id) {
          this.taskId = result.result.id;
          console.log('任务记录创建成功, ID:', this.taskId);
          
          // 显示成功提示
          uni.showToast({
            title: '照片已保存',
            icon: 'success'
          });
        } else {
          throw new Error('创建任务记录失败：没有返回ID');
        }
        
      } catch (e) {
        console.error('创建任务记录失败:', e);
        
        // 处理不同类型的错误
        if (e.message && e.message.includes('权限校验未通过')) {
          uni.showToast({
            title: '请重新登录',
            icon: 'none'
          });
          
          // 保存当前页面路径
          const currentPage = '/pages/preview/index';
          const params = `?template_id=${this.templateId}`;
          uni.setStorageSync('redirect_page', currentPage + params);
          
          setTimeout(() => {
            uni.redirectTo({
              url: '/uni_modules/uni-id-pages/pages/login/login-withpwd'
            });
          }, 1500);
          return;
        } else if (e.message && e.message.includes('database')) {
          uni.showToast({
            title: '数据库操作失败',
            icon: 'none'
          });
        } else {
          uni.showToast({
            title: '创建任务记录失败',
            icon: 'none'
          });
        }
      }
    },
    // 压缩图片方法
    async compressImage(imageUrl, maxWidth = 300, quality = 0.6) {
      return new Promise((resolve, reject) => {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        const img = new Image();
        
        img.crossOrigin = 'anonymous';
        
        img.onload = () => {
          // 计算压缩后的尺寸
          let width = img.width;
          let height = img.height;
          
          if (width > maxWidth) {
            height = Math.round((height * maxWidth) / width);
            width = maxWidth;
          }
          
          // 设置canvas尺寸
          canvas.width = width;
          canvas.height = height;
          
          // 绘制图片
          ctx.fillStyle = '#FFFFFF';
          ctx.fillRect(0, 0, width, height);
          ctx.drawImage(img, 0, 0, width, height);
          
          // 转换为base64
          const compressedImage = canvas.toDataURL('image/jpeg', quality);
          resolve(compressedImage);
        };
        
        img.onerror = (e) => {
          console.error('压缩图片失败:', e);
          reject(e);
        };
        
        img.src = imageUrl;
      });
    },
    // 更新任务记录
    async updateTaskRecord(updateData) {
      if (!this.taskId) {
        console.warn('没有找到任务ID，无法更新任务记录');
        return;
      }

      try {
        console.log('更新任务记录:', updateData);
        const db = uniCloud.database();
        const result = await db.collection('a-tasks').doc(this.taskId).update(updateData);
        console.log('任务记录更新结果:', result);
        
        // 检查更新是否成功
        if (result && result.result && result.result.updated === 1) {
          console.log('任务记录更新成功');
        } else {
          console.warn('任务记录可能未完全更新:', result);
        }
      } catch (e) {
        console.error('更新任务记录失败:', e);
      }
    },
    // 打印完成后更新打印状态
    async updatePrintStatus() {
      try {
        await this.updateTaskRecord({
          print_status: 1, // 1-已打印
          print_date: Date.now()
        });
      } catch (e) {
        console.error('更新打印状态失败:', e);
      }
    }
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.app-container {
  position: relative;
  width: 100%;
  height: 100vh;
  background-color: #000;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
  z-index: 1;
}

.preview-container {
  display: flex;
  height: 100vh;
  padding: 20px;
  position: relative;
  z-index: 2;
}

.preview-main {
  position: absolute;
  left: 0;
  top: 0;
  width: calc(100% - 160px);
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.template-preview {
  width: 100%;
  max-width: 1200px;
  height: 800px;
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.no-photo-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #cccccc;
  gap: 20px;
}

.placeholder-text {
  font-size: 18px;
  color: #999999;
}

.action-sidebar {
  width: 120px;
  padding: 20px 10px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
}

.action-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 16px 0;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  border: none;
  width: 100%;
  box-sizing: border-box;
  height: 90px;
}

.btn-icon {
  width: 30px;
  height: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 4px;
}

.icon-text {
  color: #ffffff;
  font-size: 20px;
  font-weight: bold;
}

.action-btn text {
  color: white;
  font-size: 16px;
}

.btn-complete {
  background-color: #1a73e8;
  color: white;
}

.btn-complete:active {
  background-color: #0d47a1;
}

.btn-qrcode {
  background-color: #fbbc05;
  color: white;
}

.btn-qrcode:active {
  background-color: #f9a825;
}

.btn-print {
  background-color: #34a853;
  color: white;
}

.btn-print:active {
  background-color: #2e7d32;
}

.btn-gif {
  background-color: #ea4335;
  color: white;
}

.btn-gif:active {
  background-color: #c62828;
}

.photo-info {
  margin-top: auto;
  background-color: white;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.info-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 12px;
  color: #202124;
}

.info-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 14px;
}

.info-label {
  color: #5f6368;
}

.info-value {
  color: #202124;
  font-weight: 500;
}

/* 模态框样式 */
.modal-container {
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 500px;
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #f0f0f0;
}

.modal-title {
  font-size: 18px;
  font-weight: 600;
  color: #202124;
}

.modal-close {
  cursor: pointer;
  padding: 4px;
}

.modal-body {
  padding: 20px;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 16px 20px;
  border-top: 1px solid #f0f0f0;
}

.btn {
  padding: 10px 20px;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  border: none;
}

.btn-outline {
  background-color: transparent;
  border: 1px solid #dadce0;
  color: #5f6368;
}

.btn-primary {
  background-color: #1a73e8;
  color: white;
}

/* 二维码弹窗 */
.qrcode-container {
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
}

.qrcode-image {
  width: 200px;
  height: 200px;
  border-radius: 8px;
}

.text-center {
  text-align: center;
}

.text-gray-600 {
  color: #666;
  font-size: 14px;
}

.text-gray-500 {
  color: #5f6368;
  font-size: 14px;
}

.mb-2 {
  margin-bottom: 8px;
}

.mb-4 {
  margin-bottom: 16px;
}

/* 打印预览弹窗 */
.print-preview-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.print-preview-image {
  width: 100%;
  max-width: 400px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.print-count-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-bottom: 16px;
}

.print-count-label {
  color: #5f6368;
  font-size: 16px;
}

.print-count-value {
  font-size: 20px;
  font-weight: 600;
  width: 30px;
  text-align: center;
}

.count-btn {
  width: 32px;
  height: 32px;
  background-color: #f1f3f4;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

/* GIF预览弹窗 */
.gif-preview-container {
  width: 100%;
  height: 300px;
  background-color: #f0f0f0;
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16px;
}

.gif-preview-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.gif-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.loading-spinner {
  width: 60px;
  height: 60px;
  margin-bottom: 16px;
}

.loading-text {
  font-size: 14px;
  color: #666;
}

.gif-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.gif-placeholder-text {
  font-size: 14px;
  color: #999;
  text-align: center;
}

.gif-options {
  margin-top: 16px;
}

.option-item {
  margin-bottom: 12px;
}

.option-label {
  display: block;
  font-size: 14px;
  margin-bottom: 8px;
  color: #333;
}

.speed-slider {
  display: flex;
  align-items: center;
}

.speed-label {
  font-size: 12px;
  color: #666;
  width: 30px;
}

.speed-control {
  flex: 1;
}

.btn-success {
  background-color: #28a745;
  color: #fff;
}

.btn-success:disabled {
  background-color: #8fcea0;
  color: #e0e0e0;
}

/* 打印进度弹窗 */
.print-icon-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.print-icon {
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% {
    opacity: 0.6;
    transform: scale(0.9);
  }
  50% {
    opacity: 1;
    transform: scale(1.1);
  }
  100% {
    opacity: 0.6;
    transform: scale(0.9);
  }
}

.progress-bar-container {
  width: 100%;
  height: 16px;
  background-color: #f1f3f4;
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 12px;
}

.progress-bar {
  height: 100%;
  background-color: #1a73e8;
  border-radius: 8px;
  transition: width 0.3s ease;
}
</style>