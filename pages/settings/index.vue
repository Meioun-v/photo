<template>
  <view class="app-container">
    <view class="settings-container">
      <!-- 左侧导航栏 -->
      <view class="settings-sidebar">
        <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'about'}" 
          data-section="about"
          @click="switchSection('about')"
        >
          <uni-icons type="info-filled" size="18" color="#5f6368"></uni-icons>
          <text>关于</text>
        </view>
        
        <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'general'}" 
          data-section="general"
          @click="switchSection('general')"
        >
          <uni-icons type="gear" size="18" color="#5f6368"></uni-icons>
          <text>常规</text>
        </view>
        
        <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'appearance'}" 
          data-section="appearance"
          @click="switchSection('appearance')"
        >
          <uni-icons type="color" size="18" color="#5f6368"></uni-icons>
          <text>屏幕外观</text>
        </view>
        
        <!-- <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'voice'}" 
          data-section="voice"
          @click="switchSection('voice')"
        >
          <uni-icons type="sound" size="18" color="#5f6368"></uni-icons>
          <text>语音倒数</text>
        </view> -->
        
        <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'capture'}" 
          data-section="capture"
          @click="switchSection('capture')"
        >
          <uni-icons type="camera" size="18" color="#5f6368"></uni-icons>
          <text>捕捉设置</text>
        </view>
        
        <!-- <view 
          class="settings-sidebar-item" 
          :class="{'active': activeSection === 'template'}" 
          data-section="template"
          @click="switchSection('template')"
        >
          <uni-icons type="image" size="18" color="#5f6368"></uni-icons>
          <text>模板选择</text>
        </view> -->
      </view>
      
      <!-- 右侧内容区域 -->
      <view class="settings-content">
        <view class="settings-header">
          <text class="settings-title">系统设置</text>
          <button class="settings-back-btn" @click="goBack">
            <uni-icons type="arrow-left" size="16" color="#5f6368"></uni-icons>
            <text>返回</text>
          </button>
        </view>
        
        <!-- 关于部分 -->
        <view class="settings-section" v-if="activeSection === 'about'">
          <view class="settings-section-title">关于信息</view>
          <view class="text-gray-700 mb-4">自助拍照系统是一款专为自助拍照服务设计的应用程序，支持连续拍摄、照片合成、打印和分享等功能。</view>
          
          <view class="settings-form-group">
            <view class="settings-form-label">软件版本</view>
            <view class="text-gray-900">v1.0.0</view>
          </view>
          
          <view class="settings-form-group">
            <view class="settings-form-label">联系方式</view>
            <view class="text-gray-900">电话：400-123-4567</view>
            <view class="text-gray-900">邮箱：support@photosystem.com</view>
          </view>
          
          <view class="flex justify-center mt-6">
            <image src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=https://example.com" alt="联系二维码" class="qrcode-image"></image>
          </view>
        </view>
        
        <!-- 常规设置部分 -->
        <view class="settings-section" v-if="activeSection === 'general'">
          <view class="settings-section-title">常规设置</view>
          
          <!-- 数据存储路径 -->
          <view class="settings-form-group">
            <view class="settings-form-label">数据存储路径</view>
            <view class="input-with-button">
              <input 
                type="text" 
                class="settings-form-control" 
                :value="storagePath || '请选择数据存储路径'"
                placeholder="请选择数据存储路径"
                readonly
              >
              <button class="browse-btn" @click="selectStoragePath">
                <uni-icons type="folder-open" size="16" color="#5f6368"></uni-icons>
                <text>浏览</text>
              </button>
            </view>
            
            <!-- 显示完整路径 -->
            <view class="text-sm text-gray-500 mt-1" v-if="storagePath">
              完整路径: {{storagePath}}
            </view>
            
            <!-- 存储空间信息 -->
            <view class="storage-info mt-2" v-if="storageInfo.total > 0">
              <view class="flex justify-between text-sm text-gray-600">
                <text>已使用: {{formatSize(storageInfo.used)}}</text>
                <text>可用: {{formatSize(storageInfo.free)}}</text>
              </view>
              <view class="storage-bar">
                <view 
                  class="storage-bar-fill" 
                  :style="{width: (storageInfo.used / storageInfo.total * 100) + '%'}"
                ></view>
              </view>
              <view class="text-sm text-gray-500">总容量: {{formatSize(storageInfo.total)}}</view>
            </view>
          </view>
          
          <!-- 保存按钮 -->
          <view class="settings-form-actions">
            <button class="btn btn-outline" @click="restoreDefaults('general')">恢复默认</button>
            <button class="btn btn-primary" @click="saveSettings('general')">保存设置</button>
          </view>
        </view>
        
        <!-- 屏幕外观部分 -->
        <view class="settings-section" v-if="activeSection === 'appearance'">
          <view class="settings-section-title">屏幕外观</view>
          
          <!-- 背景图片管理 -->
          <view class="settings-form-group">
            <view class="settings-form-label">背景图片选择</view>
            <view class="background-grid">
              <!-- 背景图片项 -->
              <view 
                class="bg-image-item" 
                v-for="(bg, index) in backgrounds.slice(0, 3)" 
                :key="bg.id"
                @click="selectBackground(index)"
              >
                <image :src="bg.url" alt="背景图片" class="bg-image"></image>
                <view class="bg-image-info">
                  <text class="bg-image-name">{{bg.name}}</text>
                    </view>
                <!-- 选中状态标记 -->
                <view class="bg-image-selected" v-if="bg.selected">
                  <uni-icons type="checkmarkempty" size="24" color="#ffffff"></uni-icons>
                </view>
              </view>
              
              <!-- 上传背景图片按钮 -->
              <view class="bg-image-upload" @click="uploadBackground">
                <uni-icons type="plusempty" size="32" color="#5f6368"></uni-icons>
                <text class="upload-text">上传背景</text>
              </view>
            </view>
          </view>
          
          <!-- 保存按钮 -->
          <view class="settings-form-actions">
            <button class="btn btn-outline" @click="restoreDefaults('appearance')">恢复默认</button>
            <button class="btn btn-primary" @click="saveSettings('appearance')">保存设置</button>
          </view>
        </view>
        
        <!-- 语音倒数部分 -->
        <view class="settings-section" v-if="activeSection === 'voice'">
          <view class="settings-section-title">语音倒数</view>
          
          <!-- 语音倒数开关 -->
          <view class="settings-form-group">
            <view class="flex justify-between items-center">
              <view class="settings-form-label mb-0">开启语音倒数提醒</view>
              <label class="settings-form-switch">
                <input type="checkbox" v-model="voiceEnabled">
                <view class="settings-form-slider"></view>
              </label>
            </view>
            <view class="text-sm text-gray-500 mt-1">开启后，系统将在拍照倒计时时播放语音提示</view>
          </view>
          
          <!-- 语音音色选择 -->
          <view class="settings-form-group" :class="{'opacity-50': !voiceEnabled}">
            <view class="settings-form-label">语音音色</view>
            <view class="grid grid-cols-2 gap-4 mt-2">
              <!-- 语音音色选项 -->
              <view 
                class="voice-style-item" 
                :class="{'selected': selectedVoice === 'female'}"
                @click="selectVoice('female')"
              >
                <view class="voice-style-icon">
                  <uni-icons type="person-filled" size="24" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-info">
                  <text class="voice-style-name">女声</text>
                  <text class="voice-style-desc">标准女声音色</text>
                </view>
                <view class="voice-style-check" v-if="selectedVoice === 'female'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-preview" @click.stop="previewVoice('female')">
                  <uni-icons type="play-right" size="14" color="#1a73e8"></uni-icons>
                  <text class="voice-preview-text">试听</text>
                </view>
              </view>
              
              <view 
                class="voice-style-item" 
                :class="{'selected': selectedVoice === 'male'}"
                @click="selectVoice('male')"
              >
                <view class="voice-style-icon">
                  <uni-icons type="person-filled" size="24" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-info">
                  <text class="voice-style-name">男声</text>
                  <text class="voice-style-desc">标准男声音色</text>
                </view>
                <view class="voice-style-check" v-if="selectedVoice === 'male'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-preview" @click.stop="previewVoice('male')">
                  <uni-icons type="play-right" size="14" color="#1a73e8"></uni-icons>
                  <text class="voice-preview-text">试听</text>
                </view>
              </view>
              
              <view 
                class="voice-style-item" 
                :class="{'selected': selectedVoice === 'child'}"
                @click="selectVoice('child')"
              >
                <view class="voice-style-icon">
                  <uni-icons type="person-filled" size="24" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-info">
                  <text class="voice-style-name">童声</text>
                  <text class="voice-style-desc">活泼可爱的童声</text>
                </view>
                <view class="voice-style-check" v-if="selectedVoice === 'child'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-preview" @click.stop="previewVoice('child')">
                  <uni-icons type="play-right" size="14" color="#1a73e8"></uni-icons>
                  <text class="voice-preview-text">试听</text>
                </view>
              </view>
              
              <view 
                class="voice-style-item" 
                :class="{'selected': selectedVoice === 'robot'}"
                @click="selectVoice('robot')"
              >
                <view class="voice-style-icon">
                  <uni-icons type="person-filled" size="24" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-info">
                  <text class="voice-style-name">机器人</text>
                  <text class="voice-style-desc">电子合成音色</text>
                </view>
                <view class="voice-style-check" v-if="selectedVoice === 'robot'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
                <view class="voice-style-preview" @click.stop="previewVoice('robot')">
                  <uni-icons type="play-right" size="14" color="#1a73e8"></uni-icons>
                  <text class="voice-preview-text">试听</text>
                </view>
              </view>
            </view>
          </view>
          
          <!-- 音量调节 -->
          <view class="settings-form-group" :class="{'opacity-50': !voiceEnabled}">
            <view class="settings-form-row">
              <view class="settings-form-label">音量</view>
              <view class="volume-control">
                <uni-icons type="sound" size="16" color="#5f6368"></uni-icons>
                <slider 
                  class="volume-slider" 
                  :value="volume" 
                  @change="handleVolumeChange"
                  :disabled="!voiceEnabled"
                ></slider>
                <text class="volume-value">{{volume}}%</text>
              </view>
            </view>
          </view>
          
          <!-- 保存按钮 -->
          <view class="settings-form-actions">
            <button class="btn btn-outline" @click="restoreDefaults('voice')">恢复默认</button>
            <button class="btn btn-primary" @click="saveSettings('voice')">保存设置</button>
          </view>
        </view>
        
        <!-- 捕捉设置部分 -->
        <view class="settings-section" v-if="activeSection === 'capture'">
          <view class="settings-section-title">捕捉设置</view>
          
          <!-- 倒计时时长 -->
          <view class="settings-form-group">
            <view class="settings-form-label">倒计时时长</view>
            <view class="countdown-control">
              <view class="flex-1">
                <slider 
                  :min="3" 
                  :max="8" 
                  :step="1"
                  :value="Number(countdownSeconds)" 
                  @change="handleCountdownSliderChange"
                  class="countdown-slider"
                ></slider>
              </view>
              <view class="countdown-input-group">
                <view class="countdown-number">{{ countdownSeconds }}</view>
                <text class="text-gray-700">秒</text>
              </view>
            </view>
            <view class="text-sm text-gray-500 mt-1">设置拍照前的倒计时时长（3-8秒）</view>
          </view>
          
          <!-- 闪光效果 -->
          <view class="settings-form-group">
            <view class="flex justify-between items-center">
              <view class="settings-form-label mb-0">闪光效果</view>
              <switch 
                :checked="flashEnabled" 
                @change="handleFlashChange"
                color="#1a73e8"
              />
            </view>
            <view class="text-sm text-gray-500 mt-1">开启后，拍照时会显示闪光效果</view>
          </view>
          
          <!-- 缩略图显示 -->
          <view class="settings-form-group">
            <view class="flex justify-between items-center">
              <view class="settings-form-label mb-0">缩略图显示</view>
              <switch 
                :checked="thumbnailEnabled" 
                @change="handleThumbnailChange"
                color="#1a73e8"
              />
            </view>
            <view class="text-sm text-gray-500 mt-1">开启后，拍照时会在左侧显示已拍照片的缩略图</view>
          </view>
          
          <!-- 保存按钮 -->
          <view class="settings-form-actions">
            <button class="btn btn-outline" @click="restoreDefaults('capture')">恢复默认</button>
            <button class="btn btn-primary" @click="saveSettings('capture')">保存设置</button>
          </view>
        </view>
        
        <!-- 模板选择部分 -->
        <view class="settings-section" v-if="activeSection === 'template'">
          <view class="settings-section-title">模板选择</view>
          
          <!-- 大头贴照模板 -->
          <view class="settings-form-group">
            <view class="settings-form-label">大头贴照模板</view>
            <view class="text-sm text-gray-500 mt-1 mb-4">选择适合您的大头贴照模板格式</view>
            
            <view class="grid grid-cols-2 gap-6 mt-4">
              <!-- 竖版 2x2 -->
              <view 
                class="template-item" 
                :class="{'selected': selectedTemplate === 'vertical-2x2'}"
                @click="selectTemplate('vertical-2x2')"
              >
                <view class="template-preview">
                  <view class="template-grid grid-2x2">
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                  </view>
                  <view class="template-size">2 × 2</view>
                </view>
                <view class="template-info">
                  <text class="template-name">竖版 2×2</text>
                  <text class="template-desc">4张照片竖版排列</text>
                </view>
                <view class="template-check" v-if="selectedTemplate === 'vertical-2x2'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
              </view>
              
              <!-- 竖版 2x3 -->
              <view 
                class="template-item" 
                :class="{'selected': selectedTemplate === 'vertical-2x3'}"
                @click="selectTemplate('vertical-2x3')"
              >
                <view class="template-preview">
                  <view class="template-grid grid-2x3">
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                  </view>
                  <view class="template-size">2 × 3</view>
                </view>
                <view class="template-info">
                  <text class="template-name">竖版 2×3</text>
                  <text class="template-desc">6张照片竖版排列</text>
                </view>
                <view class="template-check" v-if="selectedTemplate === 'vertical-2x3'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
              </view>
              
              <!-- 横版 2x3 -->
              <view 
                class="template-item" 
                :class="{'selected': selectedTemplate === 'horizontal-2x3'}"
                @click="selectTemplate('horizontal-2x3')"
              >
                <view class="template-preview">
                  <view class="template-grid grid-3x2">
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                    <view class="template-cell"></view>
                  </view>
                  <view class="template-size">3 × 2</view>
                </view>
                <view class="template-info">
                  <text class="template-name">横版 2×3</text>
                  <text class="template-desc">6张照片横版排列</text>
                </view>
                <view class="template-check" v-if="selectedTemplate === 'horizontal-2x3'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
              </view>
              
              <!-- 横版 1x1 -->
              <view 
                class="template-item" 
                :class="{'selected': selectedTemplate === 'horizontal-1x1'}"
                @click="selectTemplate('horizontal-1x1')"
              >
                <view class="template-preview">
                  <view class="template-grid grid-1x1">
                    <view class="template-cell"></view>
                  </view>
                  <view class="template-size">1 × 1</view>
                </view>
                <view class="template-info">
                  <text class="template-name">横版 1×1</text>
                  <text class="template-desc">单张照片横版排列</text>
                </view>
                <view class="template-check" v-if="selectedTemplate === 'horizontal-1x1'">
                  <uni-icons type="checkmarkempty" size="16" color="#1a73e8"></uni-icons>
                </view>
              </view>
            </view>
          </view>
          
          <!-- 模板预览 -->
          <view class="settings-form-group">
            <view class="settings-form-label">模板预览</view>
            <view class="template-preview-container">
              <view class="template-preview-content">
                <view class="template-preview-grid" :class="'grid-' + selectedTemplate">
                  <view class="template-preview-cell" v-for="i in getTemplateCount" :key="i">
                    <uni-icons type="person" size="30" color="#cccccc"></uni-icons>
                  </view>
                </view>
                <view class="template-preview-label">
                  {{getTemplateLabel}}
                </view>
              </view>
              <view class="template-preview-actions">
                <button class="btn btn-primary">
                  <uni-icons type="checkmarkempty" size="16" color="#ffffff"></uni-icons>
                  <text>设为默认模板</text>
                </button>
              </view>
            </view>
          </view>
          
          <!-- 保存按钮 -->
          <view class="settings-form-actions">
            <button class="btn btn-outline" @click="restoreDefaults('appearance')">恢复默认</button>
            <button class="btn btn-primary" @click="saveSettings('appearance')">保存设置</button>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'

// 添加日志函数
function log(action, message, data = {}) {
  const timestamp = new Date().toISOString();
  console.log(`[Settings ${timestamp}] ${action}:`, message, data);
  // TODO: 可以添加日志上报逻辑
}

// 添加背景图缓存配置
const BG_CACHE = {
  key: 'app_background_images',
  cloudKey: 'cloud_backgrounds',
  localKey: 'local_backgrounds',
  selectedKey: 'selected_background'
};

export default {
  components: {
    uniIcons
  },
  data() {
    return {
      activeSection: 'about',
      settings: {
        about: [],
        general: [],
        appearance: [],
        voice: [],
        capture: []
      },
      // 语音相关
      voiceEnabled: true,
      selectedVoice: 'female',
      volume: 80,
      // 捕捉设置相关
      countdownSeconds: 5,
      flashEnabled: true,
      thumbnailEnabled: true,
      // 模板相关
      selectedTemplate: 'vertical-2x2',
      // 背景相关
      backgrounds: [],
      // 加载状态
      isLoading: true,
      // 是否有未保存的更改
      hasUnsavedChanges: false,
      // 存储路径相关
      storagePath: localStorage.getItem('storage_local_path') || '',
      storageInfo: {
        used: 0,
        free: 0,
        total: 0
      },
      directoryHandle: null,
      // 添加背景缓存相关数据
      backgroundCache: {
        cloudBackgrounds: [],
        localBackgrounds: [],
        selectedBackground: null
      }
    }
  },
  computed: {
    getTemplateCount() {
      switch (this.selectedTemplate) {
        case 'vertical-2x2':
          return 4;
        case 'vertical-2x3':
        case 'horizontal-2x3':
          return 6;
        case 'horizontal-1x1':
          return 1;
        default:
          return 4;
      }
    },
    getTemplateLabel() {
      switch (this.selectedTemplate) {
        case 'vertical-2x2':
          return '竖版 2×2';
        case 'vertical-2x3':
          return '竖版 2×3';
        case 'horizontal-2x3':
          return '横版 2×3';
        case 'horizontal-1x1':
          return '横版 1×1';
        default:
          return '竖版 2×2';
      }
    }
  },
  async onLoad() {
    log('Page Load', '设置页面加载');
    await this.loadSettings();
  },
  methods: {
    // 加载设置
    async loadSettings() {
      try {
        log('Load Settings', '开始加载所有设置');
        this.isLoading = true;
        
        // 1. 从云端获取about和appearance数据
        const db = uniCloud.database();
        const { result } = await db.collection('a-settings')
          .where({
            group: {
              $in: ['about', 'appearance']
            }
          })
          .get();
          
        log('Cloud Settings Loaded', '从云端加载about和appearance设置成功', {
          settingsCount: result.data.length
        });
        
        // 处理云端数据
        result.data.forEach(item => {
          if (!this.settings[item.group]) {
            this.settings[item.group] = [];
          }
          this.settings[item.group].push(item);
        });
        
        // 2. 从本地缓存获取其他设置
        const cachedSettings = localStorage.getItem('app_settings');
        if (cachedSettings) {
          const localSettings = JSON.parse(cachedSettings);
          // 合并本地设置，但不覆盖云端数据
          Object.keys(localSettings).forEach(group => {
            if (group !== 'about' && group !== 'appearance') {
              this.settings[group] = localSettings[group];
            }
          });
          
          log('Local Settings Loaded', '从本地缓存加载其他设置成功', {
            groups: Object.keys(localSettings).filter(g => g !== 'about' && g !== 'appearance')
          });
        } else {
          // 如果本地没有缓存，初始化默认值
          const defaultSettings = {
            voice: [
              { key: 'countdown_voice_model', value: 'female', group: 'voice' }
            ],
            capture: [
              { key: 'capture_countdown_seconds', value: 5, group: 'capture' },
              { key: 'capture_flash_screen', value: true, group: 'capture' },
              { key: 'capture_thumbnail_show', value: true, group: 'capture' }
            ]
          };
          
          Object.keys(defaultSettings).forEach(group => {
            this.settings[group] = defaultSettings[group];
          });
          
          // 保存默认设置到本地
          localStorage.setItem('app_settings', JSON.stringify(defaultSettings));
          log('Default Settings Created', '创建默认本地设置');
        }
        
        // 初始化各项设置值
        this.initializeSettings();
        
      } catch (error) {
        log('Load Settings Error', '加载设置失败', { error: error.message });
        console.error('加载设置失败:', error);
        uni.showToast({
          title: '加载设置失败',
          icon: 'none'
        });
      } finally {
        this.isLoading = false;
      }
    },
    
    // 初始化设置值
    initializeSettings() {
      log('Initialize Settings', '开始初始化设置值');
      
      // 初始化语音设置
      const voiceSettings = this.settings.voice;
      if (voiceSettings) {
        const voiceModel = voiceSettings.find(s => s.key === 'countdown_voice_model');
        if (voiceModel) {
          this.selectedVoice = voiceModel.value;
          log('Voice Settings', '初始化语音设置', { selectedVoice: this.selectedVoice });
        }
      }
      
      // 初始化捕捉设置
      const captureSettings = this.settings.capture;
      if (captureSettings && captureSettings.length > 0) {
        // 倒计时秒数
        const countdown = captureSettings.find(s => s.key === 'capture_countdown_seconds');
        if (countdown) {
          // 确保转换为数字
          let value = Number(countdown.value);
          if (isNaN(value)) value = 5;
          if (value < 3) value = 3;
          if (value > 8) value = 8;
          this.countdownSeconds = value;
          
          // 强制更新UI（确保显示正确的值）
          this.$nextTick(() => {
            this.countdownSeconds = value;
          });
        }
        
        // 闪光效果
        const flash = captureSettings.find(s => s.key === 'capture_flash_screen');
        if (flash) {
          // 确保是布尔值
          this.flashEnabled = flash.value === true;
        }
        
        // 缩略图显示
        const thumbnail = captureSettings.find(s => s.key === 'capture_thumbnail_show');
        if (thumbnail) {
          // 确保是布尔值
          this.thumbnailEnabled = thumbnail.value === true;
        }
        
        log('Capture Settings', '初始化捕捉设置', {
          countdownSeconds: this.countdownSeconds,
          flashEnabled: this.flashEnabled,
          thumbnailEnabled: this.thumbnailEnabled
        });
      } else {
        // 如果没有捕捉设置，设置默认值
        this.countdownSeconds = 5;
        this.flashEnabled = true;
        this.thumbnailEnabled = true;
        log('Capture Settings', '使用默认捕捉设置');
      }
      
      // 初始化背景设置
      this.loadBackgrounds();
      
      // 初始化存储路径
      const savedPath = localStorage.getItem('storage_local_path');
      if (savedPath) {
        this.storagePath = savedPath;
        log('Storage Settings', '初始化存储路径', { storagePath: this.storagePath });
        this.updateStorageInfo();
      }
      
      log('Settings Initialized', '设置初始化完成');
    },
    
    // 选择存储路径
    async selectStoragePath() {
      try {
        log('Select Storage Path', '开始选择存储路径');
        
        // 使用 showDirectoryPicker API 选择目录
        const dirHandle = await window.showDirectoryPicker({
          mode: 'readwrite'
        });
        
        // 获取完整路径
        let path = '';
        try {
          // 尝试方法1: 使用 resolve
          const relativePath = await dirHandle.resolve();
          if (relativePath && relativePath.length > 0) {
            path = relativePath.join('/');
            log('Path Resolution', '使用resolve方法获取路径成功', { path });
          }
        } catch (e) {
          log('Path Resolution Warning', 'resolve方法获取路径失败', { error: e.message });
          
          try {
            // 尝试方法2: 使用 getDirectory
            const rootDir = await navigator.storage.getDirectory();
            const fullPath = await dirHandle.resolve(rootDir);
            if (fullPath && fullPath.length > 0) {
              path = fullPath.join('/');
              log('Path Resolution', '使用getDirectory方法获取路径成功', { path });
            }
          } catch (e) {
            log('Path Resolution Warning', 'getDirectory方法获取路径失败', { error: e.message });
            
            try {
              // 尝试方法3: 递归获取父目录
              let currentHandle = dirHandle;
              const pathParts = [dirHandle.name];
              
              while (true) {
                try {
                  const parentHandle = await currentHandle.getParent();
                  if (!parentHandle) break;
                  
                  pathParts.unshift(parentHandle.name);
                  currentHandle = parentHandle;
                } catch (e) {
                  break;
                }
              }
              
              path = pathParts.join('/');
              log('Path Resolution', '使用递归方法获取路径成功', { path });
            } catch (e) {
              log('Path Resolution Warning', '递归方法获取路径失败', { error: e.message });
              // 如果所有方法都失败,使用基本信息
              path = dirHandle.name;
            }
          }
        }
        
        if (!path) {
          path = dirHandle.name;
          log('Path Resolution Warning', '所有方法都失败,使用目录名称', { path });
        }
        
        // 确保路径不为空且格式正确
        if (path) {
          // 标准化路径格式
          path = path.replace(/\\/g, '/');
          // 移除开头的斜杠
          path = path.replace(/^\/+/, '');
          // 移除结尾的斜杠
          path = path.replace(/\/+$/, '');
        }
        
        log('Storage Path Selected', '存储路径选择成功', { path });
        
        // 更新存储路径并保存到localStorage
        this.storagePath = path;
        localStorage.setItem('storage_local_path', path);
        this.hasUnsavedChanges = false;
        
        // 保存目录句柄以供后续使用
        this.directoryHandle = dirHandle;
        
        // 更新存储信息
        await this.updateStorageInfo();
        
        // 显示成功提示
        uni.showToast({
          title: '已选择存储路径',
          icon: 'success'
        });
        
      } catch (error) {
        log('Select Storage Path Error', '选择目录失败', { error: error.message });
        console.error('选择目录失败:', error);
        if (error.name !== 'AbortError') {
          uni.showToast({
            title: '选择目录失败',
            icon: 'none'
          });
        }
      }
    },
    
    // 更新存储信息
    async updateStorageInfo() {
      try {
        log('Update Storage Info', '开始更新存储信息');
        
        if (this.directoryHandle) {
          // 如果有目录句柄，尝试获取目录大小信息
          let totalSize = 0;
          let fileCount = 0;
          
          try {
            // 遍历目录获取文件大小
            for await (const entry of this.directoryHandle.values()) {
              if (entry.kind === 'file') {
                try {
                  const file = await entry.getFile();
                  if (file && file.size) {
                    totalSize += file.size;
                    fileCount++;
                  }
                  log('File Size', `文件 ${file.name} 大小`, { size: this.formatSize(file.size) });
                } catch (e) {
                  log('File Access Warning', `无法访问文件 ${entry.name}`, { error: e.message });
                }
              }
            }
            
            log('Directory Stats', '目录统计信息', { 
              totalSize: this.formatSize(totalSize), 
              fileCount,
              rawSize: totalSize 
            });
            
            // 更新存储信息，确保至少有1GB空间
            const minSpace = 1024 * 1024 * 1024; // 1GB
            this.storageInfo = {
              used: totalSize,
              total: Math.max(totalSize * 2, minSpace),
              free: Math.max(minSpace - totalSize, minSpace / 2)
            };
            
          } catch (e) {
            log('Directory Size Warning', '无法获取目录大小', { error: e.message });
            throw e;
          }
        } else if ('storage' in navigator && 'estimate' in navigator.storage) {
          // 如果无法获取目录信息，使用浏览器存储估算
          const estimate = await navigator.storage.estimate();
          this.storageInfo = {
            used: estimate.usage || 0,
            total: estimate.quota || 1024 * 1024 * 1024,
            free: (estimate.quota || 1024 * 1024 * 1024) - (estimate.usage || 0)
          };
          
          log('Browser Storage Stats', '浏览器存储信息', {
            used: this.formatSize(this.storageInfo.used),
            free: this.formatSize(this.storageInfo.free),
            total: this.formatSize(this.storageInfo.total)
          });
        }
        
        log('Storage Info Updated', '存储信息更新完成', {
          used: this.formatSize(this.storageInfo.used),
          free: this.formatSize(this.storageInfo.free),
          total: this.formatSize(this.storageInfo.total)
        });
        
      } catch (error) {
        log('Update Storage Info Error', '更新存储信息失败', { error: error.message });
        console.error('获取存储信息失败:', error);
        // 设置默认值
        this.storageInfo = {
          used: 0,
          total: 1024 * 1024 * 1024, // 1GB
          free: 1024 * 1024 * 1024 // 1GB
        };
      }
    },
    
    // 格式化文件大小
    formatSize(bytes) {
      if (bytes === 0) return '0 B';
      const k = 1024;
      const sizes = ['B', 'KB', 'MB', 'GB', 'TB'];
      const i = Math.floor(Math.log(bytes) / Math.log(k));
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    },
    
    // 保存设置
    async saveSettings(group) {
      try {
        log('Save Settings', `开始保存${group}组设置`);
        
        // 根据不同设置组更新本地缓存
        switch (group) {
          case 'voice':
            this.settings.voice = this.settings.voice.map(item => {
              if (item.key === 'countdown_voice_model') {
                return {
                  ...item,
                  value: this.selectedVoice,
                  update_date: new Date()
                };
              }
              return item;
            });
            break;
            
          case 'capture':
            this.settings.capture = this.settings.capture.map(item => {
              switch (item.key) {
                case 'capture_countdown_seconds':
                  return { ...item, value: this.countdownSeconds, update_date: new Date() };
                case 'capture_flash_screen':
                  return { ...item, value: this.flashEnabled, update_date: new Date() };
                case 'capture_thumbnail_show':
                  return { ...item, value: this.thumbnailEnabled, update_date: new Date() };
                default:
                  return item;
              }
            });
            break;
            
          case 'appearance':
            // 获取选中的背景图片
            const selectedBg = this.backgrounds.find(bg => bg.selected);
            if (selectedBg) {
              // 处理背景图URL
              let processedUrl = selectedBg.url;
              if (selectedBg.url.startsWith('cloud://')) {
                try {
                  const result = await uniCloud.getTempFileURL({
                    fileList: [selectedBg.url]
                  });
                  if (result.fileList && result.fileList[0] && result.fileList[0].tempFileURL) {
                    processedUrl = result.fileList[0].tempFileURL;
                  }
                } catch (error) {
                  log('Background URL Process Error', '处理背景图URL失败', { error: error.message });
                }
              }
              
              // 更新本地缓存
              uni.setStorageSync(BG_CACHE.selectedKey, processedUrl);
              
              // 保存背景设置到本地
              const localSettings = JSON.parse(localStorage.getItem('app_settings') || '{}');
              if (!localSettings.appearance) {
                localSettings.appearance = [];
              }
              
              // 更新或添加背景设置
              const bgSettingIndex = localSettings.appearance.findIndex(s => s.key === 'home_background');
              const bgSetting = {
                key: 'home_background',
                value: this.backgrounds.map(bg => ({
                  id: bg.id,
                  name: bg.name,
                  url: bg.url,
                  selected: bg.selected
                })),
                group: 'appearance',
                update_date: new Date()
              };
              
              if (bgSettingIndex >= 0) {
                localSettings.appearance[bgSettingIndex] = bgSetting;
              } else {
                localSettings.appearance.push(bgSetting);
              }
              
              localStorage.setItem('app_settings', JSON.stringify(localSettings));
              log('Background Settings Saved', '背景设置已保存到本地');
            }
            break;
        }
        
        // 只保存非云端数据到本地缓存
        if (group !== 'about') {
          const localSettings = {
            ...JSON.parse(localStorage.getItem('app_settings') || '{}'),
            [group]: this.settings[group]
          };
          localStorage.setItem('app_settings', JSON.stringify(localSettings));
        }
        
        this.hasUnsavedChanges = false;
        log('Settings Saved', `${group}组设置保存成功`);
        
        uni.showToast({
          title: '保存成功',
          icon: 'success'
        });
        
      } catch (error) {
        log('Save Settings Error', '保存设置失败', { 
          group,
          error: error.message 
        });
        
        console.error('保存设置失败:', error);
        uni.showToast({
          title: '保存失败，请重试',
          icon: 'none'
        });
      }
    },
    
    // 恢复默认设置
    async restoreDefaults(group) {
      try {
        log('Restore Defaults', `开始恢复${group}组默认设置`);
        
        if (group === 'general') {
          // 清除存储路径设置
          this.storagePath = '';
          localStorage.removeItem('storage_local_path');
          this.directoryHandle = null;
          this.storageInfo = {
            used: 0,
            free: 0,
            total: 0
          };
          log('Storage Path Reset', '存储路径已重置');
        } else if (group === 'about' || group === 'appearance') {
          // 从云端重新获取数据
          const db = uniCloud.database();
          const { result } = await db.collection('a-settings')
            .where({
              group: group
            })
            .get();
          
          this.settings[group] = result.data;
          
          // 清除背景图缓存并设置默认值
          uni.removeStorageSync(BG_CACHE.selectedKey);
          uni.setStorageSync(BG_CACHE.selectedKey, BG_CACHE.localKey);
          
          log('Background Cache Reset', '背景图缓存已重置为默认值');
        } else {
          // 本地设置恢复默认值
          const defaultSettings = {
            voice: [
              { key: 'countdown_voice_model', value: 'female', group: 'voice' }
            ],
            capture: [
              { key: 'capture_countdown_seconds', value: 5, group: 'capture' },
              { key: 'capture_flash_screen', value: true, group: 'capture' },
              { key: 'capture_thumbnail_show', value: true, group: 'capture' }
            ]
          };
          
          if (defaultSettings[group]) {
            this.settings[group] = defaultSettings[group];
            // 更新本地缓存
            const localSettings = JSON.parse(localStorage.getItem('app_settings') || '{}');
            localSettings[group] = defaultSettings[group];
            localStorage.setItem('app_settings', JSON.stringify(localSettings));
          }
        }
        
        // 重新初始化设置
        this.initializeSettings();
        
        log('Defaults Restored', `${group}组默认设置已恢复`);
        
        uni.showToast({
          title: '已恢复默认设置',
          icon: 'success'
        });
        
      } catch (error) {
        log('Restore Defaults Error', '恢复默认设置失败', {
          group,
          error: error.message
        });
        
        console.error('恢复默认设置失败:', error);
        uni.showToast({
          title: '恢复失败，请重试',
          icon: 'none'
        });
      }
    },
    
    // 切换设置组
    switchSection(section) {
      // 如果有未保存的更改，提示用户
      if (this.hasUnsavedChanges) {
        uni.showModal({
          title: '未保存的更改',
          content: '您有未保存的更改，是否继续？',
          success: (res) => {
            if (res.confirm) {
      this.activeSection = section;
              this.hasUnsavedChanges = false;
            }
          }
        });
      } else {
        this.activeSection = section;
      }
    },
    
    // 返回按钮
    goBack() {
      // 如果有未保存的更改，提示用户
      if (this.hasUnsavedChanges) {
        uni.showModal({
          title: '未保存的更改',
          content: '您有未保存的更改，是否继续？',
          success: (res) => {
            if (res.confirm) {
      uni.navigateTo({
        url: '/pages/start/index'
      });
            }
          }
        });
      } else {
        uni.navigateTo({
          url: '/pages/start/index'
        });
      }
    },
    
    // 语音相关方法
    selectVoice(voice) {
      if (!this.voiceEnabled) return;
      this.selectedVoice = voice;
      this.hasUnsavedChanges = true;
    },
    
    previewVoice(voice) {
      if (!this.voiceEnabled) return;
      // TODO: 实现语音预览功能
      uni.showToast({
        title: `正在试听${voice}音色`,
        icon: 'none'
      });
    },
    
    handleVolumeChange(e) {
      this.volume = e.detail.value;
      this.hasUnsavedChanges = true;
    },
    
    // 加载背景图片
    async loadBackgrounds() {
      try {
        log('Load Backgrounds', '开始加载背景图片');
        
        // 1. 从本地缓存加载数据
        const cachedData = uni.getStorageSync(BG_CACHE.key);
        if (cachedData) {
          this.backgroundCache = JSON.parse(cachedData);
          this.backgrounds = [
            ...this.backgroundCache.cloudBackgrounds,
            ...this.backgroundCache.localBackgrounds
          ];
          
          // 恢复选中状态
          if (this.backgroundCache.selectedBackground) {
            const selectedIndex = this.backgrounds.findIndex(
              bg => bg.id === this.backgroundCache.selectedBackground
            );
            if (selectedIndex >= 0) {
              this.backgrounds[selectedIndex].selected = true;
            }
          }
          
          log('Cache Loaded', '从缓存加载背景图片成功', {
            totalCount: this.backgrounds.length,
            cloudCount: this.backgroundCache.cloudBackgrounds.length,
            localCount: this.backgroundCache.localBackgrounds.length
          });
        }
        
        // 2. 从云端获取最新数据
        const db = uniCloud.database();
        const { result } = await db.collection('a-settings')
          .where({
            group: 'appearance',
            key: 'home_background'
          })
          .get();
          
        if (result.data && result.data[0] && result.data[0].value) {
          // 更新云端背景列表
          this.backgroundCache.cloudBackgrounds = result.data[0].value.map(bg => ({
            ...bg,
            source: 'cloud'
          }));
          
          // 合并本地和云端背景
          this.backgrounds = [
            ...this.backgroundCache.cloudBackgrounds,
            ...this.backgroundCache.localBackgrounds
          ];
          
          // 保存到缓存
          this.saveBackgroundCache();
          
          log('Cloud Backgrounds Updated', '更新云端背景图片成功', {
            cloudCount: this.backgroundCache.cloudBackgrounds.length
          });
        }
        
      } catch (error) {
        log('Load Backgrounds Error', '加载背景图片失败', { error: error.message });
        console.error('加载背景图片失败:', error);
      }
    },
    
    // 上传背景图片
    async uploadBackground() {
      try {
        // 选择图片
        const res = await uni.chooseImage({
        count: 1,
          sizeType: ['original', 'compressed'],
          sourceType: ['album']
        });
        
        if (!res.tempFiles || res.tempFiles.length === 0) return;
        
        const tempFile = res.tempFiles[0];
        const fileName = tempFile.name || `background_${Date.now()}.jpg`;
        
        // 创建本地背景对象
        const newBackground = {
          id: `local_${Date.now()}`,
          name: fileName.split('.')[0],
          url: tempFile.path,
          source: 'local',
          selected: false
        };
        
        // 添加到本地背景列表
        this.backgroundCache.localBackgrounds.push(newBackground);
        
        // 更新显示列表
        this.backgrounds = [
          ...this.backgroundCache.cloudBackgrounds,
          ...this.backgroundCache.localBackgrounds
        ];
        
        // 保存到缓存
        this.saveBackgroundCache();
        
        log('Background Uploaded', '上传背景图片成功', {
          fileName,
          backgroundId: newBackground.id
        });
        
          uni.showToast({
          title: '上传成功',
            icon: 'success'
          });
        
      } catch (error) {
        log('Upload Background Error', '上传背景图片失败', { error: error.message });
        console.error('上传背景图片失败:', error);
        uni.showToast({
          title: '上传失败',
          icon: 'none'
        });
      }
    },
    
    // 保存背景缓存
    saveBackgroundCache() {
      try {
        uni.setStorageSync(BG_CACHE.key, JSON.stringify(this.backgroundCache));
        log('Background Cache Saved', '保存背景缓存成功');
      } catch (error) {
        log('Save Background Cache Error', '保存背景缓存失败', { error: error.message });
      }
    },
    
    // 修改选择背景方法
    async selectBackground(index) {
      // 更新选中状态
      this.backgrounds.forEach((bg, i) => {
        bg.selected = i === index;
      });
      
      // 获取选中的背景图片
      const selectedBg = this.backgrounds[index];
      
      // 更新缓存中的选中状态
      this.backgroundCache.selectedBackground = selectedBg.id;
      
      // 处理背景图URL
      let processedUrl = selectedBg.url;
      if (selectedBg.source === 'cloud' && selectedBg.url.startsWith('cloud://')) {
        try {
          const result = await uniCloud.getTempFileURL({
            fileList: [selectedBg.url]
          });
          if (result.fileList && result.fileList[0] && result.fileList[0].tempFileURL) {
            processedUrl = result.fileList[0].tempFileURL;
        }
      } catch (error) {
          log('Background URL Process Error', '处理背景图URL失败', { error: error.message });
        }
      }
      
      // 保存到本地缓存
      try {
        uni.setStorageSync(BG_CACHE.selectedKey, processedUrl);
        this.saveBackgroundCache();
        
        log('Background Selected', '选择背景图片成功', {
          selectedId: selectedBg.id,
          backgroundName: selectedBg.name,
          processedUrl
        });
      } catch (error) {
        log('Save Selected Background Error', '保存选中背景失败', { error: error.message });
      }
      
      this.hasUnsavedChanges = true;
    },
    
    // 处理倒计时滑块变更
    handleCountdownSliderChange(e) {
      const value = Number(e.detail.value);
      if (!isNaN(value)) {
        this.countdownSeconds = value;
        this.hasUnsavedChanges = true;
        log('Countdown Changed', '倒计时时长已更改', { value });
      }
    },
    
    // 处理倒计时输入
    handleCountdownInput(e) {
      let value = Number(e.detail.value);
      if (isNaN(value)) value = 5;
      if (value < 3) value = 3;
      if (value > 8) value = 8;
      this.countdownSeconds = value;
      this.hasUnsavedChanges = true;
      log('Countdown Input', '倒计时输入已更改', { value });
    },
    
    // 处理闪光效果变更
    handleFlashChange(e) {
      this.flashEnabled = e.detail.value;
      this.hasUnsavedChanges = true;
      log('Flash Changed', '闪光效果已更改', { enabled: this.flashEnabled });
    },
    
    // 处理缩略图显示变更
    handleThumbnailChange(e) {
      this.thumbnailEnabled = e.detail.value;
      this.hasUnsavedChanges = true;
      log('Thumbnail Changed', '缩略图显示已更改', { enabled: this.thumbnailEnabled });
    },
    
    // 模板相关方法
    selectTemplate(template) {
      this.selectedTemplate = template;
      this.hasUnsavedChanges = true;
    },
  }
}
</script>

<style>
body {
  background-color: #f8f9fa;
  margin: 0;
  overflow: hidden;
  height: 100vh;
}

.app-container {
  width: 100%;
  height: 100vh;
}

.settings-container {
  display: flex;
  height: 100vh;
}

.settings-sidebar {
  width: 240px;
  background-color: white;
  border-right: 1px solid #dadce0;
  padding: 20px 0;
  overflow-y: auto;
}

.settings-sidebar-item {
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 500;
  color: #5f6368;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  gap: 12px;
}

.settings-sidebar-item:active {
  background-color: rgba(26, 115, 232, 0.1);
  color: #1a73e8;
}

.settings-sidebar-item.active {
  background-color: rgba(26, 115, 232, 0.2);
  color: #1a73e8;
  border-left: 3px solid #1a73e8;
}

.settings-sidebar-item.active uni-icons {
  color: #1a73e8 !important;
}

.settings-content {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  position: relative;
}

.settings-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 1px solid #dadce0;
  position: relative;
}

.settings-title {
  font-size: 24px;
  font-weight: 600;
  color: #202124;
  margin-left: 120px;
  text-align: center;
  flex: 1;
}

.settings-back-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #5f6368;
  background-color: white;
  border: 1px solid #dadce0;
  cursor: pointer;
  transition: all 0.2s ease;
  position: absolute;
  left: 0;
  top: -10px;
}

.settings-back-btn:active {
  background-color: #f8f9fa;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.settings-section {
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 24px;
  margin-bottom: 24px;
}

.settings-section-title {
  font-size: 18px;
  font-weight: 600;
  color: #202124;
  margin-bottom: 16px;
}

.settings-form-group {
  margin-bottom: 20px;
}

.settings-form-label {
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: #5f6368;
  margin-bottom: 8px;
}

.settings-form-control {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #dadce0;
  border-radius: 8px;
  font-size: 14px;
  transition: border-color 0.2s ease;
  box-sizing: border-box;
}

.settings-form-control:focus {
  border-color: #1a73e8;
  outline: none;
}

.text-gray-700 {
  color: #5f6368;
  font-size: 14px;
  line-height: 1.5;
}

.text-gray-900 {
  color: #202124;
  font-size: 14px;
  line-height: 1.5;
}

.mb-4 {
  margin-bottom: 16px;
}

.mt-6 {
  margin-top: 24px;
}

.mt-2 {
  margin-top: 8px;
}

.flex {
  display: flex;
}

.justify-center {
  justify-content: center;
}

.justify-between {
  justify-content: space-between;
}

.qrcode-image {
  width: 200px;
  height: 200px;
  border-radius: 8px;
}

.input-with-button {
  display: flex;
  gap: 8px;
}

.input-with-button .settings-form-control {
  flex: 1;
}

.browse-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background-color: #f8f9fa;
  border: 1px solid #dadce0;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #5f6368;
  cursor: pointer;
  transition: all 0.2s ease;
}

.browse-btn:active {
  background-color: #f1f3f4;
}

.storage-info {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 12px;
  margin-top: 8px;
}

.text-sm {
  font-size: 12px;
}

.storage-bar {
  height: 8px;
  background-color: #e0e0e0;
  border-radius: 4px;
  margin: 8px 0;
  overflow: hidden;
}

.storage-bar-fill {
  height: 100%;
  background-color: #1a73e8;
  border-radius: 4px;
  width: 35%;
}

.settings-form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

.btn {
  padding: 10px 20px;
  border-radius: 6px;
  font-size: 14px;
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

.background-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-top: 16px;
}

.bg-image-item {
  position: relative;
  width: 100%;
  height: 160px;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.bg-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.bg-image-info {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 8px;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.bg-image-name {
  color: white;
  font-size: 14px;
}

.settings-form-switch {
  position: relative;
  display: inline-block;
  width: 48px;
  height: 24px;
}

.settings-form-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.settings-form-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 24px;
}

.settings-form-slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .settings-form-slider {
  background-color: #1a73e8;
}

input:checked + .settings-form-slider:before {
  transform: translateX(24px);
}

.voice-style-item {
  position: relative;
  border: 1px solid #dadce0;
  border-radius: 8px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.voice-style-item:active {
  border-color: #1a73e8;
  background-color: rgba(26, 115, 232, 0.05);
}

.voice-style-item.selected {
  border-color: #1a73e8;
}

.voice-style-icon {
  width: 40px;
  height: 40px;
  background-color: rgba(26, 115, 232, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
}

.voice-style-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.voice-style-name {
  font-size: 14px;
  font-weight: 500;
  color: #202124;
}

.voice-style-desc {
  font-size: 12px;
  color: #5f6368;
}

.voice-style-check {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: rgba(26, 115, 232, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.voice-style-preview {
  position: absolute;
  bottom: 8px;
  right: 8px;
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  border-radius: 4px;
  background-color: rgba(26, 115, 232, 0.1);
  cursor: pointer;
}

.voice-preview-text {
  font-size: 12px;
  color: #1a73e8;
}

.volume-control {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 12px;
}

.volume-slider {
  flex: 1;
}

.volume-value {
  width: 48px;
  text-align: right;
  font-size: 14px;
  color: #5f6368;
}

.opacity-50 {
  opacity: 0.5;
  pointer-events: none;
}

.items-center {
  align-items: center;
}

.mb-0 {
  margin-bottom: 0;
}

.countdown-control {
  display: flex;
  align-items: center;
  gap: 12px;
}

.countdown-slider {
  flex: 1;
}

.countdown-input-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.text-center {
  text-align: center;
}

.flex-1 {
  flex: 1;
}

.template-item {
  position: relative;
  border: 1px solid #dadce0;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.2s ease;
}

.template-item:active {
  border-color: #1a73e8;
  background-color: rgba(26, 115, 232, 0.05);
}

.template-item.selected {
  border-color: #1a73e8;
}

.template-preview {
  position: relative;
  height: 160px;
  background-color: #f8f9fa;
  display: flex;
  align-items: center;
  justify-content: center;
}

.template-grid {
  position: absolute;
  inset: 12px;
  display: grid;
  gap: 4px;
}

.grid-2x2 {
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
}

.grid-2x3 {
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 1fr);
}

.grid-3x2 {
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
}

.grid-1x1 {
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
}

.template-cell {
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.template-size {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 18px;
  font-weight: bold;
  color: #cccccc;
}

.template-info {
  padding: 12px;
}

.template-name {
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: #202124;
  margin-bottom: 4px;
}

.template-desc {
  display: block;
  font-size: 12px;
  color: #5f6368;
}

.template-check {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: rgba(26, 115, 232, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.template-preview-container {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 16px;
}

.template-preview-content {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 16px;
  position: relative;
}

.template-preview-grid {
  display: grid;
  gap: 8px;
  aspect-ratio: 3/4;
}

.grid-vertical-2x2 {
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
}

.grid-vertical-2x3 {
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 1fr);
}

.grid-horizontal-2x3 {
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  aspect-ratio: 4/3;
}

.grid-horizontal-1x1 {
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
  aspect-ratio: 4/3;
}

.template-preview-cell {
  background-color: #f8f9fa;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.template-preview-label {
  position: absolute;
  bottom: 16px;
  left: 16px;
  background-color: white;
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 14px;
  font-weight: 500;
  color: #5f6368;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.template-preview-actions {
  display: flex;
  justify-content: center;
  margin-top: 16px;
}

.grid {
  display: grid;
}

.grid-cols-2 {
  grid-template-columns: repeat(2, 1fr);
}

.gap-6 {
  gap: 24px;
}

.mb-4 {
  margin-bottom: 16px;
}

.loading-more {
  text-align: center;
  padding: 20px 0;
  color: #5f6368;
  font-size: 14px;
}

.no-more {
  text-align: center;
  padding: 20px 0;
  color: #9aa0a6;
  font-size: 14px;
}

.bg-image-selected {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background-color: #1a73e8;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.bg-image-upload {
  width: 100%;
  height: 160px;
  border-radius: 8px;
  border: 2px dashed #dadce0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.bg-image-upload:hover {
  border-color: #1a73e8;
  background-color: rgba(26, 115, 232, 0.05);
}

.upload-text {
  margin-top: 8px;
  font-size: 14px;
  color: #5f6368;
}

.countdown-number {
  font-size: 16px;
  font-weight: 500;
  color: #202124;
  min-width: 30px;
  text-align: center;
}
</style>
