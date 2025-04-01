<template>
  <view class="app-container" :style="backgroundStyle">
    <view class="template-header">
      <view class="header-btn btn-cancel" @click="goBack">
        <uni-icons type="arrow-left" size="18" color="#ffffff"></uni-icons>
        <text>返回</text>
      </view>
      
      <view class="template-title">选择模板</view>
      
      <view class="header-btn btn-confirm" :class="{'btn-disabled': !selectedTemplate}" @click="confirmSelection">
        <uni-icons type="checkmarkempty" size="18" color="#ffffff"></uni-icons>
        <text>确认</text>
      </view>
    </view>
    
    <view class="template-content">
      <!-- 加载中状态 -->
      <view class="loading-container" v-if="loading">
        <uni-load-more status="loading" :contentText="{ contentdown: '加载中...' }"></uni-load-more>
      </view>
      
      <!-- 无数据状态 -->
      <view class="empty-container" v-else-if="templates.length === 0">
        <view class="empty-message">
          <uni-icons type="info" size="32" color="#999999"></uni-icons>
          <text>暂无可用模板</text>
        </view>
      </view>
      
      <!-- 模板列表 -->
      <view class="template-carousel-container" v-else>
        <!-- 左侧控制按钮 -->
        <!-- 
        <view class="carousel-control prev" @click.stop="scrollToPrev" :style="{opacity: currentPage === 0 ? '0.5' : '1', pointerEvents: currentPage === 0 ? 'none' : 'auto'}">
          <uni-icons type="left" size="18" color="#333333"></uni-icons>
        </view>
        -->
        
        <!-- 模板轮播 -->
        <scroll-view 
          class="template-carousel" 
          scroll-x="true" 
          @scroll="onCarouselScroll" 
          :scroll-left="scrollPosition" 
          scroll-with-animation="true" 
          id="templateCarousel"
          enhanced="true"
          show-scrollbar="false"
        >
          <view class="carousel-content">
            <view 
              class="template-item" 
              v-for="(template, index) in templates" 
              :key="template._id"
              :class="{'selected': selectedTemplateId === template._id}"
              @click="selectTemplate(template)"
            >
              <view class="template-preview">
                <!-- 使用缩略图 -->
                <image
                  v-if="template.thumbnail_url" 
                  :src="template.thumbnail_url" 
                  class="template-thumbnail"
                  mode="aspectFit"
                ></image>
                <!-- 无缩略图时显示默认布局 -->
                <view v-else :class="getTemplateClass(template)">
                  <view class="cell" v-for="i in template.photo_count" :key="i"></view>
                </view>
              </view>
              <view class="template-info">
                <view class="template-name">{{template.name}}</view>
                <view class="template-desc">{{template.photo_count}}张照片</view>
              </view>
            </view>
          </view>
        </scroll-view>
        
        <!-- 右侧控制按钮 -->
        <!--
        <view class="carousel-control next" @click.stop="scrollToNext" :style="{opacity: currentPage === totalPages - 1 ? '0.5' : '1', pointerEvents: currentPage === totalPages - 1 ? 'none' : 'auto'}">
          <uni-icons type="right" size="18" color="#333333"></uni-icons>
        </view>
        -->
        
        <!-- 分页指示器 -->
        <view class="carousel-indicators">
          <view 
            class="indicator" 
            v-for="i in totalPages" 
            :key="i-1"
            :class="{'active': currentPage === i-1}"
            @click="scrollToPage(i-1)"
          ></view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'
import { getAppBackground } from '@/uni_modules/uni-id-pages/pages/login/login-withpwd.vue';

export default {
  components: {
    uniIcons
  },
  data() {
    return {
      templates: [],
      selectedTemplateId: null,
      selectedTemplate: null,
      currentPage: 0,
      totalPages: 0,
      templatesPerPage: 3,
      scrollPosition: 0,
      carouselWidth: 0,
      loading: true,
      backgroundImage: '/static/login-bg.jpg' // 默认背景图
    }
  },
  computed: {
    backgroundStyle() {
      if (this.backgroundImage.startsWith('#')) {
        return { backgroundColor: this.backgroundImage };
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
  async onLoad() {
    try {
      // 加载背景图
      this.backgroundImage = await getAppBackground();
      
      // 加载模板数据
      this.loadTemplates();
    } catch (e) {
      console.error('页面初始化出错:', e);
    }
  },
  onReady() {
    // 获取轮播容器宽度
    const query = uni.createSelectorQuery().in(this);
    query.select('#templateCarousel').boundingClientRect(data => {
      if (data) {
        this.carouselWidth = data.width;
        this.updateTotalPages();
      }
    }).exec();
  },
  methods: {
    async loadTemplates() {
      try {
        this.loading = true;
        
        // 从云数据库查询模板
        const userInfo = uniCloud.getCurrentUserInfo();
        const uid = userInfo && userInfo.uid;
        
        console.log('当前用户ID:', uid);
        const db = uniCloud.database();
        const templatesCollection = db.collection('a-templates');
        
        // 仅查询启用状态的模板并按排序号排序
        const { result } = await templatesCollection.where({
          status: 1,
          operator_id:uid
        }).orderBy('sort', 'asc').get();
        
        console.log('模板查询结果:', result);
        
        if (result && result.data) {
          this.templates = result.data;
          
          // 处理缩略图路径
          this.templates = await Promise.all(this.templates.map(async template => {
            // 处理可能的云存储路径
            if (template.thumbnail_url && template.thumbnail_url.startsWith('cloud://')) {
              try {
                const res = await uniCloud.getTempFileURL({
                  fileList: [template.thumbnail_url]
                });
                if (res.fileList && res.fileList[0] && res.fileList[0].tempFileURL) {
                  template.thumbnail_url = res.fileList[0].tempFileURL;
                }
              } catch (e) {
                console.error('获取缩略图临时链接失败:', e);
              }
            }
            return template;
          }));
        } else {
          this.templates = [];
        }
      } catch (e) {
        console.error('加载模板失败:', e);
        uni.showToast({
          title: '加载模板失败，请重试',
          icon: 'none'
        });
        this.templates = [];
      } finally {
        this.loading = false;
        this.updateTotalPages();
      }
    },
    updateTotalPages() {
      if (this.templates.length > 0 && this.carouselWidth > 0) {
        this.totalPages = Math.max(1, Math.ceil(this.templates.length / this.templatesPerPage));
        console.log(`更新分页: 共${this.templates.length}个模板，每页${this.templatesPerPage}个，共${this.totalPages}页`);
      }
    },
    // 使用模板的实际尺寸比例生成样式类名
    getTemplateClass(template) {
      // 根据模板的尺寸比例返回不同的样式类
      const { width, height, photo_count } = template;
      
      if (!width || !height) {
        // 没有尺寸信息时使用照片数量来决定类名
        if (photo_count <= 3) {
          return 'horizontal-template-1';
        } else if (photo_count <= 6) {
          return 'horizontal-template-2';
        } else {
          return 'vertical-template-3';
        }
      }
      
      // 使用尺寸比例
      const ratio = width / height;
      
      if (ratio > 1) {
        // 横向模板
        return 'horizontal-template-1';
      } else if (ratio === 1) {
        // 正方形模板
        return 'square-template';
      } else {
        // 纵向模板
        return 'vertical-template-3';
      }
    },
    selectTemplate(template) {
      this.selectedTemplateId = template._id;
      this.selectedTemplate = template;
    },
    scrollToPage(page) {
      if (page < 0 || page >= this.totalPages) return;
      
      console.log(`滚动到第${page + 1}页`);
      this.currentPage = page;
      // 计算每个模板的宽度（包括margin）
      const templateWidth = this.getTemplateWidth();
      // 计算滚动位置
      this.scrollPosition = page * templateWidth * this.templatesPerPage;
      console.log(`滚动位置: ${this.scrollPosition}px`);
    },
    scrollToPrev() {
      console.log('点击上一页按钮');
      this.scrollToPage(this.currentPage - 1);
    },
    scrollToNext() {
      console.log('点击下一页按钮');
      this.scrollToPage(this.currentPage + 1);
    },
    // 获取模板项的宽度（包括margin）
    getTemplateWidth() {
      // 默认值，可能需要调整
      const defaultWidth = 330; // 300px + 30px margin-right
      
      // 尝试获取实际宽度
      try {
        const item = document.querySelector('.template-item');
        if (item) {
          const style = window.getComputedStyle(item);
          const width = parseFloat(style.width);
          const marginRight = parseFloat(style.marginRight);
          return width + marginRight;
        }
      } catch (e) {
        console.error('获取模板宽度失败:', e);
      }
      
      return defaultWidth;
    },
    onCarouselScroll(e) {
      const scrollLeft = e.detail.scrollLeft;
      if (this.carouselWidth > 0) {
        // 计算每个模板的宽度（包括margin）
        const templateWidth = this.getTemplateWidth();
        const itemsPerPage = this.templatesPerPage;
        
        // 计算当前页码
        const newPage = Math.round(scrollLeft / (templateWidth * itemsPerPage));
        
        if (newPage !== this.currentPage) {
          console.log(`滚动更新页码：从第${this.currentPage + 1}页到第${newPage + 1}页`);
          this.currentPage = newPage;
        }
      }
    },
    confirmSelection() {
      if (!this.selectedTemplate) return;
      
      // 将选中的模板信息存储到本地
      uni.setStorageSync('selected_template', this.selectedTemplate);
      
      // 跳转到拍照页面
      uni.navigateTo({
        url: `/pages/capture/index?template_id=${this.selectedTemplate._id}&photo_count=${this.selectedTemplate.photo_count}`
      });
    },
    goBack() {
      uni.navigateBack();
    }
  }
}
</script>

<style>
.app-container {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.loading-container, .empty-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.empty-message {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  color: #999999;
}

.template-thumbnail {
  width: 100%;
  height: 100%;
  border-radius: 8px;
}

.template-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 40px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
}

.template-title {
  font-size: 24px;
  font-weight: 600;
}

.header-btn {
  padding: 12px 24px;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
  transition: all 0.2s ease;
}

.btn-cancel {
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
}

.btn-cancel:active {
  background-color: rgba(255, 255, 255, 0.3);
}

.btn-confirm {
  background-color: #34a853;
  color: white;
}

.btn-confirm:active {
  background-color: #2d9249;
}

.btn-disabled {
  background-color: rgba(52, 168, 83, 0.5);
  opacity: 0.7;
}

.template-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  overflow: hidden;
}

.template-carousel-container {
  width: 100%;
  max-width: 1200px;
  position: relative;
}

.template-carousel {
  display: flex;
  white-space: nowrap;
  padding: 20px 0;
  width: 100%;
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE and Edge */
  overflow-x: auto;
}

.template-carousel::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}

.carousel-content {
  display: inline-flex;
  flex-wrap: nowrap;
}

.template-item {
  display: inline-block;
  background-color: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
  position: relative;
  min-width: 300px;
  max-width: 300px;
  margin-right: 30px;
  flex-shrink: 0;
}

.template-item:active {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.template-item.selected {
  border: 3px solid #1a73e8;
}

.template-item.selected::after {
  content: '\2714';
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: #1a73e8;
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 12px;
}

.template-preview {
  height: 200px;
  background-color: #f5f5f5;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.template-info {
  padding: 15px;
  text-align: center;
}

.template-name {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin-bottom: 5px;
}

.template-desc {
  font-size: 14px;
  color: #666;
}

/* 模板预览样式 */
.horizontal-template-1 {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 5px;
  width: 90%;
  height: 90%;
}

.horizontal-template-1 .cell {
  background-color: rgba(255, 255, 255, 0.3);
  border: 1px dashed #ccc;
}

.horizontal-template-2 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 5px;
  width: 90%;
  height: 90%;
}

.horizontal-template-2 .cell {
  background-color: rgba(255, 255, 255, 0.3);
  border: 1px dashed #ccc;
}

.vertical-template-3 {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 5px;
  width: 70%;
  height: 90%;
}

.vertical-template-3 .cell {
  background-color: rgba(255, 255, 255, 0.3);
  border: 1px dashed #ccc;
}

/* 滑动控制按钮 */
.carousel-control {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;
  cursor: pointer;
}

.carousel-control:active {
  background-color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  transform: translateY(-50%) scale(1.1);
}

.carousel-control.prev {
  left: -10px;
}

.carousel-control.next {
  right: -10px;
}

/* 分页指示器 */
.carousel-indicators {
  display: flex;
  justify-content: center;
  gap: 8px;
  margin-top: 20px;
}

.indicator {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.3);
  transition: all 0.2s ease;
}

.indicator.active {
  background-color: white;
  transform: scale(1.2);
}
</style> 