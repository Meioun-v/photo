<template>
  <view class="app-container">
    <!-- 顶部标题栏 -->
    <view class="page-header">
      <view class="header-left">
        <button class="back-button" type="default" @click="goBack">
          <uni-icons type="arrow-left" size="16" color="#333"></uni-icons>
          <text class="back-text">返回</text>
        </button>
        <view class="page-title">历史拍摄记录</view>
      </view>
      <view class="search-container">
        <view class="search-box">
          <uni-icons type="search" size="18" color="#999"></uni-icons>
          <input class="search-input" type="text" v-model="query" @confirm="search" placeholder="搜索拍摄记录..." />
        </view>
        <button class="search-button" type="primary" size="mini" @click="search">搜索</button>
      </view>
    </view>
    
    <!-- 图片预览弹窗 -->
    <uni-popup ref="previewPopup" type="dialog">
      <view class="preview-popup">
        <view class="preview-header">
          <text class="preview-title">图片预览</text>
          <button class="close-button" @click="closePreview">
            <uni-icons type="close" size="20" color="#333"></uni-icons>
          </button>
        </view>
        <swiper class="preview-swiper" :current="currentPreviewIndex" @change="handleSwiperChange">
          <swiper-item v-for="(img, index) in previewImages" :key="index">
            <image :src="img" mode="aspectFit" class="preview-image" />
          </swiper-item>
        </swiper>
        <view class="preview-footer">
          <text class="preview-counter">{{ currentPreviewIndex + 1 }} / {{ previewImages.length }}</text>
        </view>
      </view>
    </uni-popup>
    
    <!-- 主要内容区域 -->
    <view class="content-container">
      <unicloud-db ref="udb" 
        collection="a-tasks"
        field="_id,template_id,photo_count,photos,result_images,gif_url,create_date,operator_id" 
        :where="'operator_id == $cloudEnv_uid'"
        :getone="false"
        page-data="replace"
        :orderby="orderby || 'create_date desc'" 
        :getcount="true" 
        :page-size="options.pageSize" 
        :page-current="options.pageCurrent"
        v-slot:default="{data,pagination,loading,error,options}" 
        :options="options" 
        @load="onqueryload">
        
        <!-- 加载状态 -->
        <view v-if="loading" class="loading-container">
          <view class="loader"></view>
          <text class="loading-text">加载中...</text>
        </view>
        
        <!-- 错误状态 -->
        <view v-else-if="error" class="error-container">
          <uni-icons type="error" size="45" color="#f56c6c"></uni-icons>
          <text class="error-text">{{error.message || '加载失败'}}</text>
          <button class="retry-button" size="mini" @click="loadData()">重试</button>
        </view>
        
        <!-- 空数据状态 -->
        <view v-else-if="!data || data.length === 0" class="empty-container">
          <image src="/static/empty-box.png" class="empty-image"></image>
          <text class="empty-text">暂无历史记录</text>
          <text class="empty-tip">完成拍摄后的记录将会显示在这里</text>
        </view>
        
        <!-- 数据表格 -->
        <view v-else class="table-container">
          <uni-table ref="table" :loading="loading" :emptyText="error.message || '没有更多数据'" border stripe>
            <uni-tr class="table-header">
              <uni-th align="center" width="120px" sortable @sort-change="sortChange($event, '_id')">任务编号</uni-th>
              <uni-th align="center" width="180px" sortable @sort-change="sortChange($event, 'create_date')">创建时间</uni-th>
              <uni-th align="center">拍摄照片</uni-th>
              <uni-th align="center">成品图片</uni-th>
              <uni-th align="center" width="200px">操作</uni-th>
            </uni-tr>
            
            <uni-tr v-for="(item,index) in data" :key="index" class="table-row">
              <uni-td align="center">
                <view class="task-id">{{item._id}}</view>
              </uni-td>
              <uni-td align="center">
                <view class="date-cell">
                  <uni-dateformat 
                    :threshold="[0, 0]" 
                    :date="item.create_date" 
                    format="yyyy-MM-dd hh:mm"
                  ></uni-dateformat>
                </view>
              </uni-td>
              <uni-td align="center">
                <view class="image-preview-container">
                  <view v-if="item.photos && item.photos.length" class="image-grid">
                    <view v-for="(photo, photoIndex) in (item.photos.slice(0, 4))" :key="photoIndex" class="image-item">
                      <image 
                        :src="photo.thumbnail_url || photo.photo_url" 
                        mode="aspectFill" 
                        class="thumbnail"
                        @click="previewImage(item.photos, photoIndex, 'photo_url')"
                      />
                    </view>
                    <view v-if="item.photos.length > 4" class="more-images">
                      <text>+{{item.photos.length - 4}}</text>
                    </view>
                  </view>
                  <text v-else class="no-image-text">暂无图片</text>
                </view>
              </uni-td>
              <uni-td align="center">
                <view class="image-preview-container">
                  <view v-if="item.result_images && item.result_images.length" class="image-grid">
                    <view v-for="(image, imageIndex) in item.result_images" :key="imageIndex" class="image-item">
                      <image 
                        :src="image.thumbnail_url || image.image_url" 
                        mode="aspectFill" 
                        class="thumbnail"
                        @click="previewImage(item.result_images, imageIndex, 'image_url')"
                      />
                    </view>
                  </view>
                  <text v-else class="no-image-text">暂无图片</text>
                </view>
              </uni-td>
              <uni-td align="center">
                <view class="action-buttons-wrapper">
                  <view class="action-buttons">
                    <button 
                      v-if="item.result_images && item.result_images.length" 
                      @click="handlePrint(item.result_images[0])" 
                      class="action-btn print-btn"
                      hover-class="btn-hover"
                    >
                      <uni-icons type="paperplane" size="16" color="#fff"></uni-icons>
                      <text>打印</text>
                    </button>
                    <button 
                      @click="downloadSingleTask(item)" 
                      class="action-btn download-btn"
                      hover-class="btn-hover"
                    >
                      <uni-icons type="download" size="16" color="#fff"></uni-icons>
                      <text>下载</text>
                    </button>
                    <button 
                      @click="confirmDelete(item._id)" 
                      class="action-btn delete-btn"
                      hover-class="btn-hover"
                    >
                      <uni-icons type="trash" size="16" color="#fff"></uni-icons>
                      <text>删除</text>
                    </button>
                  </view>
                </view>
              </uni-td>
            </uni-tr>
          </uni-table>
          
          <!-- 分页控件 -->
          <view class="pagination-box">
            <uni-pagination 
              show-icon 
              :page-size="pagination.size" 
              v-model="pagination.current" 
              :total="pagination.count" 
              @change="onPageChanged" 
            />
          </view>
        </view>
      </unicloud-db>
    </view>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'
import uniPopup from '@/uni_modules/uni-popup/components/uni-popup/uni-popup.vue'
import uniPopupDialog from '@/uni_modules/uni-popup/components/uni-popup-dialog/uni-popup-dialog.vue'
import uniTable from '@/uni_modules/uni-table/components/uni-table/uni-table.vue'
import uniTr from '@/uni_modules/uni-table/components/uni-tr/uni-tr.vue'
import uniTh from '@/uni_modules/uni-table/components/uni-th/uni-th.vue'
import uniTd from '@/uni_modules/uni-table/components/uni-td/uni-td.vue'
import uniPagination from '@/uni_modules/uni-pagination/components/uni-pagination/uni-pagination.vue'
import uniDateformat from '@/uni_modules/uni-dateformat/components/uni-dateformat/uni-dateformat.vue'
import JSZip from 'jszip';
import printService from '@/utils/printService.js';

const db = uniCloud.database()
// 表查询配置
const dbOrderBy = 'create_date desc' // 默认按创建时间倒序
const dbSearchFields = ['_id'] // 搜索字段
// 分页配置
const pageSize = 20
const pageCurrent = 1

const orderByMapping = {
  "ascending": "asc",
  "descending": "desc"
}

export default {
  components: {
    uniIcons,
    uniPopup,
    uniPopupDialog,
    uniTable,
    uniTr,
    uniTh,
    uniTd,
    uniPagination,
    uniDateformat
  },
  data() {
    return {
      query: '',
      where: '',
      orderby: dbOrderBy,
      orderByFieldName: "",
      options: {
        pageSize,
        pageCurrent
      },
      tasksToDelete: null,
      previewImages: [], // 当前预览的图片列表
      currentPreviewIndex: 0, // 当前预览的图片索引
      templateMap: {} // 用于缓存模板ID到名称的映射
    }
  },
  onLoad() {
    console.log('历史照片页面加载');
  },
  methods: {
    // 查询数据
    onqueryload(data) {
      console.log('查询数据加载完成:', data);
      
      if (!data || data.length === 0) {
        console.log('没有查询到数据');
        return;
      }
      
      // 加载所有出现的模板ID，并获取模板名称
      try {
        const templateIds = [...new Set(data.map(item => item.template_id))].filter(id => id);
        // if (templateIds.length > 0) {
        //   this.loadTemplateNames(templateIds);
        // }
      } catch (e) {
        console.error('处理模板ID时出错:', e);
      }
    },
    
    // 加载模板名称
    async loadTemplateNames(templateIds) {
      console.log('开始加载模板名称:', templateIds);
      
      // 过滤出尚未加载的模板ID
      const needToLoad = templateIds.filter(id => !this.templateMap[id]);
      
      if (needToLoad.length === 0) {
        console.log('没有需要加载的模板');
        return;
      }
      
      try {
        const db = uniCloud.database();
        const res = await db.collection('a-templates')
          .where(`_id in ${JSON.stringify(needToLoad)}`)
          .field('_id,name')
          .get();

        console.log('模板查询结果:', res);
        
        // 更新模板映射
        if (res && res.result && res.result.data) {
          const templates = res.result.data;
          templates.forEach(template => {
            this.$set(this.templateMap, template._id, template.name || template._id);
          });
          console.log('模板映射已更新:', this.templateMap);
        } else {
          console.warn('未找到匹配的模板信息');
        }
      } catch (e) {
        console.error('加载模板名称失败:', e);
        // 如果加载失败，使用ID作为名称
        needToLoad.forEach(id => {
          this.$set(this.templateMap, id, id);
        });
      }
    },
    
    // 获取模板名称
    getTemplateName(templateId) {
      return this.templateMap[templateId] || templateId
    },
    
    // 搜索
    search() {
      const query = this.query.trim()
      if (!query) {
        this.where = ''
      } else {
        // 构建搜索条件
        const queryRe = new RegExp(query, 'i')
        this.where = dbSearchFields.map(name => queryRe + '.test(' + name + ')').join(' || ')
      }
      
      this.$nextTick(() => {
        this.loadData()
      })
    },
    
    loadData(clear = true) {
      this.$refs.udb.loadData({
        clear
      })
    },
    
    onPageChanged(e) {
      this.$refs.udb.loadData({
        current: e.current
      })
    },
    
    // 预览图片
    previewImage(images, index, urlField = 'photo_url') {
      // 提取所有图片URL
      this.previewImages = images.map(img => img[urlField])
      this.currentPreviewIndex = index
      this.$refs.previewPopup.open()
    },
    
    // 滑动切换预览图片
    handleSwiperChange(e) {
      this.currentPreviewIndex = e.detail.current
    },
    
    // 关闭预览
    closePreview() {
      this.$refs.previewPopup.close()
    },
    
    // 返回上一页
    goBack() {
      uni.navigateTo({
        url: '/pages/start/index'
      });
    },
    
    // 排序变化
    sortChange(e, name) {
      console.log('排序变化:', e, name);
      
      try {
        this.orderByFieldName = name;
        if (e.order) {
          this.orderby = name + ' ' + orderByMapping[e.order];
        } else {
          this.orderby = 'create_date desc'; // 默认排序
        }
        
        this.$nextTick(() => {
          if (this.$refs.udb) {
            this.$refs.udb.loadData();
          } else {
            console.error('udb引用不存在');
          }
        });
      } catch (e) {
        console.error('处理排序变化时出错:', e);
        uni.showToast({
          title: '排序操作出错',
          icon: 'none'
        });
      }
    },
    
    // 确认删除
    confirmDelete(id) {
      console.log('确认删除:', id);
      this.tasksToDelete = id;
      
      // 显示确认对话框
      uni.showModal({
        title: '确认删除',
        content: '确定要删除此任务及其所有照片吗？此操作无法撤销。',
        confirmText: '删除',
        confirmColor: '#f56c6c',
        cancelText: '取消',
        success: (res) => {
          if (res.confirm) {
            this.deleteTask();
          } else {
            this.tasksToDelete = null;
          }
        }
      });
    },
    
    // 执行删除
    async deleteTask() {
      if (!this.tasksToDelete) {
        console.log('没有要删除的任务');
        return;
      }
      
      console.log('开始删除任务:', this.tasksToDelete);
      
      try {
        uni.showLoading({
          title: '删除中...'
        });
        
        const db = uniCloud.database();
        const result = await db.collection('a-tasks').doc(this.tasksToDelete).remove();
        
        console.log('删除结果:', result);
        
        uni.showToast({
          title: '删除成功',
          icon: 'success'
        });
        
        // 重新加载数据
        this.loadData();
      } catch (e) {
        console.error('删除失败:', e);
        uni.showToast({
          title: '删除失败: ' + (e.message || '未知错误'),
          icon: 'none'
        });
      } finally {
        uni.hideLoading();
        this.tasksToDelete = null;
      }
    },
    
    // 打印照片
    async handlePrint(photo) {
      console.log('打印照片对象详情:', JSON.stringify(photo));
      
      if (!photo || !photo.image_url) {
        uni.showToast({
          title: '没有可打印的图片',
          icon: 'none'
        });
        return;
      }
      
      try {
        // 显示加载提示
        uni.showLoading({
          title: '正在打印...',
          mask: true
        });
        
        // 直接使用已导入的printService
        // 执行打印 - 只打印1份
        await printService.printMultipleCopies(photo.image_url, 1);
        
        // 隐藏加载提示
        uni.hideLoading();
        
        // 显示成功提示
        uni.showToast({
          title: '打印完成！',
          icon: 'success'
        });
        
      } catch (e) {
        console.error('打印出错:', e);
        uni.hideLoading();
        uni.showToast({
          title: e.message || '打印失败',
          icon: 'none'
        });
      }
    },
    
    // 下载单个任务
    async downloadSingleTask(task) {
      console.log('下载单个任务:', task);
      
      if (!task) {
        uni.showToast({
          title: '没有可下载的任务',
          icon: 'none'
        });
        return;
      }
      
      try {
        uni.showLoading({
          title: '准备下载...'
        });
        
        // 调用云函数下载图片
        const { result } = await uniCloud.callFunction({
          name: 'downloadTaskImages',
          data: {
            taskId: task._id
          }
        });
        
        console.log('云函数返回结果:', result); // 打印完整返回结果用于调试
        
        if (!result.success) {
          throw new Error(result.message || '下载失败');
        }
        
        if (!result.images || result.images.length === 0) {
          throw new Error('没有可下载的图片');
        }
        
        // 创建zip
        const zip = new JSZip();
        const originalFolder = zip.folder('原始照片');
        const resultFolder = zip.folder('成品图片');
        
        // 是否需要直接从URL下载
        const isUrlOnly = result.urlsOnly === true;
        
        if (isUrlOnly) {
          console.log('使用URL直接下载模式');
          uni.showLoading({
            title: '从URL下载图片...'
          });
          
          const totalImages = result.images.length;
          let completedImages = 0;
          let failedImages = 0;
          
          // 创建一个进度更新函数
          const updateProgress = () => {
            completedImages++;
            const progress = Math.floor((completedImages / totalImages) * 100);
            uni.showLoading({
              title: `下载中: ${progress}%`
            });
          };
          
          // 使用Promise.all并行下载所有图片
          try {
            const downloadPromises = result.images.map(img => {
              return new Promise((resolve, reject) => {
                console.log(`从URL下载图片: ${img.url}`);
                
                // 使用fetch API下载图片
                fetch(img.url)
                  .then(response => {
                    if (!response.ok) {
                      throw new Error('图片下载失败: ' + response.status);
                    }
                    return response.blob();
                  })
                  .then(blob => {
                    const folder = img.type === 'original' ? originalFolder : resultFolder;
                    folder.file(img.filename, blob);
                    updateProgress();
                    resolve();
                  })
                  .catch(error => {
                    console.error('下载图片失败:', error, img.url);
                    failedImages++;
                    updateProgress();
                    resolve(); // 即使失败也resolve，确保所有promise都完成
                  });
              });
            });
            
            // 等待所有下载完成
            await Promise.all(downloadPromises);
            
            console.log(`图片下载完成，成功: ${completedImages - failedImages}, 失败: ${failedImages}`);
            
            if (completedImages - failedImages === 0) {
              throw new Error('所有图片下载失败');
            }
          } catch (e) {
            console.error('下载图片过程出错:', e);
            throw new Error('下载过程中出错: ' + e.message);
          }
        } else {
          // 添加图片到zip
          let addedImages = 0;
          for (const image of result.images) {
            try {
              if (!image.data) {
                console.warn(`图片 ${image.filename} 没有数据`);
                continue;
              }
              
              console.log(`处理图片: ${image.filename}, 数据类型: ${typeof image.data}, 是否为字符串: ${typeof image.data === 'string'}`);
              
              let imageData;
              
              // 处理不同类型的数据返回
              if (typeof image.data === 'string') {
                // 检查是否是已经编码的URL格式 (data:image/jpeg;base64,...)
                if (image.data.startsWith('data:')) {
                  // 提取base64部分
                  const base64Data = image.data.split(',')[1];
                  console.log(`图片 ${image.filename} 是data URL格式，提取base64部分`);
                  imageData = base64Data;
                } else {
                  // 假设是纯base64字符串
                  imageData = image.data;
                }
                
                // 验证是否是合法的Base64编码
                if (!/^[A-Za-z0-9+/=]+$/.test(imageData)) {
                  console.warn(`图片 ${image.filename} 的数据不是有效的Base64格式，尝试直接处理`);
                  
                  // 如果有URL，尝试使用URL进行下载
                  if (image.url && typeof image.url === 'string' && image.url.startsWith('http')) {
                    console.log(`图片 ${image.filename} 有URL，尝试后续处理`);
                    continue;
                  }
                  
                  // 如果是图片URL，尝试直接通过URL添加
                  if (image.data.startsWith('http')) {
                    console.log(`图片 ${image.filename} 是URL，跳过处理`);
                    continue;
                  }
                }
              } else if (image.data && image.data.type === 'binary' && Array.isArray(image.data.data)) {
                // 处理云函数返回的二进制数据数组
                console.log(`图片 ${image.filename} 是二进制数据数组，长度:`, image.data.data.length);
                
                try {
                  // 将数组转换回Uint8Array
                  const uint8Array = new Uint8Array(image.data.data);
                  const folder = image.type === 'original' ? originalFolder : resultFolder;
                  folder.file(image.filename, uint8Array);
                  addedImages++;
                  image.processed = true;
                  console.log(`已添加图片(二进制数组): ${image.filename}`);
                  continue;
                } catch (binaryError) {
                  console.error(`处理二进制数组失败: ${image.filename}`, binaryError);
                  // 如果失败，继续尝试其他方法
                }
              } else if (image.data instanceof Uint8Array || Array.isArray(image.data)) {
                // 如果返回的是二进制数组
                console.log(`图片 ${image.filename} 是数组类型，长度: ${image.data.length}`);
                const folder = image.type === 'original' ? originalFolder : resultFolder;
                folder.file(image.filename, image.data);
                addedImages++;
                console.log(`已添加图片(数组): ${image.filename}`);
                continue;
              } else if (typeof image.data === 'object') {
                // 处理对象类型的数据 (可能是Buffer对象)
                console.log(`图片 ${image.filename} 是对象类型，尝试提取数据`);
                
                try {
                  let dataToUse;
                  
                  // 检查是否是Buffer对象
                  if (image.data.type === 'Buffer' && Array.isArray(image.data.data)) {
                    console.log(`图片 ${image.filename} 是Buffer对象，转换为Uint8Array`);
                    dataToUse = new Uint8Array(image.data.data);
                  } else if (image.data instanceof ArrayBuffer || ArrayBuffer.isView(image.data)) {
                    // 已经是ArrayBuffer或ArrayBufferView (如 Uint8Array)
                    console.log(`图片 ${image.filename} 是ArrayBuffer或ArrayBufferView`);
                    dataToUse = image.data;
                  } else if (image.data.byteLength !== undefined) {
                    // 可能是ArrayBuffer或类似的二进制数据
                    console.log(`图片 ${image.filename} 可能是ArrayBuffer，转换为Uint8Array`);
                    dataToUse = new Uint8Array(image.data);
                  } else if (image.data.data && (image.data.data instanceof ArrayBuffer || ArrayBuffer.isView(image.data.data))) {
                    // 某些情况下数据可能被包裹在data属性中
                    console.log(`图片 ${image.filename} 数据在data属性中`);
                    dataToUse = image.data.data;
                  } else {
                    // 尝试JSON序列化后再处理
                    try {
                      const jsonStr = JSON.stringify(image.data);
                      console.log(`图片 ${image.filename} 是其他对象类型，JSON序列化长度: ${jsonStr.length}`);
                      
                      // 如果对象很小，可能不是有效的图片数据
                      if (jsonStr.length < 100) {
                        console.warn(`图片 ${image.filename} 序列化后数据太小，可能不是有效图片数据`);
                        
                        // 如果有URL，记录起来后续处理
                        if (image.url) {
                          console.log(`图片 ${image.filename} 有URL，尝试后续处理`);
                          image.processed = false;
                        }
                        continue;
                      }
                      
                      // 尝试使用TextEncoder
                      const encoder = new TextEncoder();
                      dataToUse = encoder.encode(jsonStr);
                    } catch (jsonError) {
                      console.error(`图片 ${image.filename} JSON序列化失败:`, jsonError);
                      
                      // 如果有URL，我们可以回退到使用URL
                      if (image.url) {
                        console.log(`图片 ${image.filename} 将使用URL下载`);
                        image.processed = false;
                        continue;
                      }
                      
                      throw jsonError;
                    }
                  }
                  
                  if (dataToUse && dataToUse.byteLength > 0) {
                    const folder = image.type === 'original' ? originalFolder : resultFolder;
                    folder.file(image.filename, dataToUse);
                    addedImages++;
                    image.processed = true;
                    console.log(`已添加图片(对象转换): ${image.filename}`);
                  } else {
                    console.warn(`图片 ${image.filename} 转换后没有有效数据`);
                    // 标记为未处理，后续可能通过URL处理
                    image.processed = false;
                  }
                } catch (objError) {
                  console.error(`处理对象类型图片数据失败: ${image.filename}`, objError);
                  // 标记为未处理，后续可能通过URL处理
                  image.processed = false;
                }
                
                continue;
              } else {
                console.warn(`图片 ${image.filename} 数据类型不支持: ${typeof image.data}`);
                continue;
              }
              
              try {
                // 尝试将base64转换为二进制
                const binaryString = atob(imageData);
                const bytes = new Uint8Array(binaryString.length);
                for (let i = 0; i < binaryString.length; i++) {
                  bytes[i] = binaryString.charCodeAt(i);
                }
                
                const folder = image.type === 'original' ? originalFolder : resultFolder;
                folder.file(image.filename, bytes);
                addedImages++;
                console.log(`已添加图片: ${image.filename}`);
              } catch (e) {
                console.error(`Base64解码失败: ${image.filename}`, e);
                // 如果URL可用，将单独处理
              }
            } catch (e) {
              console.error(`处理图片 ${image.filename} 失败:`, e);
              // 继续处理其他图片
            }
          }
          
          // 处理URL列表 - 如果有图片带URL但未成功处理
          const urlImages = result.images.filter(img => 
            (img.url && typeof img.url === 'string' && img.url.startsWith('http')) &&
            (img.processed === false || img.processed === undefined || img.fallback === true) // 未被处理过或标记为fallback
          );
          
          if (urlImages.length > 0) {
            console.log('尝试从URL下载未成功处理的图片:', urlImages.length);
            
            const totalUrls = urlImages.length;
            let completedUrls = 0;
            
            // 更新进度
            const updateUrlProgress = () => {
              completedUrls++;
              uni.showLoading({
                title: `下载图片: ${Math.floor((completedUrls / totalUrls) * 100)}%`
              });
            };
            
            const urlPromises = urlImages.map(img => {
              return new Promise((resolve, reject) => {
                fetch(img.url)
                  .then(response => {
                    if (!response.ok) {
                      throw new Error('图片下载失败: ' + response.status);
                    }
                    return response.blob();
                  })
                  .then(blob => {
                    const folder = img.type === 'original' ? originalFolder : resultFolder;
                    folder.file(img.filename, blob);
                    addedImages++;
                    updateUrlProgress();
                    resolve();
                  })
                  .catch(error => {
                    console.error('下载图片失败:', error, img.url);
                    updateUrlProgress();
                    resolve(); // 即使失败也resolve
                  });
              });
            });
            
            // 等待所有URL下载完成
            await Promise.all(urlPromises);
          }
          
          if (addedImages === 0) {
            throw new Error('没有成功添加任何图片');
          }
        }
        
        // 生成ZIP文件
        console.log('开始生成ZIP文件...');
        const content = await zip.generateAsync({
          type: 'blob',
          compression: 'DEFLATE',
          compressionOptions: {
            level: 6
          }
        });
        console.log('ZIP文件生成完成，大小:', content.size);
        
        // 下载ZIP文件
        const url = URL.createObjectURL(content);
        const link = document.createElement('a');
        const filename = `拍照任务_${task._id}.zip`;
        link.download = filename;
        link.href = url;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        URL.revokeObjectURL(url);
        
        uni.hideLoading();
        uni.showToast({
          title: '下载完成',
          icon: 'success'
        });
      } catch (e) {
        console.error('下载失败:', e);
        uni.hideLoading();
        uni.showToast({
          title: '下载失败: ' + e.message,
          icon: 'none'
        });
      }
    }
  }
}
</script>

<style>
.app-container {
  width: 100%;
  min-height: 100vh;
  background-color: #f8f8f8;
  display: flex;
  flex-direction: column;
}

/* 页面头部 */
.page-header {
  background-color: #fff;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 10;
}

.header-left {
  display: flex;
  align-items: center;
}

.back-button {
  display: flex;
  align-items: center;
  background: transparent;
  border: none;
  padding: 8px 12px;
  margin-right: 15px;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.back-text {
  margin-left: 4px;
  font-size: 14px;
}

.page-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.search-container {
  display: flex;
  align-items: center;
  margin-left: 20px;
}

.search-box {
  display: flex;
  align-items: center;
  background-color: #f5f5f5;
  border-radius: 20px;
  padding: 6px 12px;
  margin-right: 10px;
  width: 240px;
  border: 1px solid #eee;
  transition: all 0.3s;
}

.search-box:focus-within {
  background-color: #fff;
  border-color: #007aff;
  box-shadow: 0 0 0 2px rgba(0,122,255,0.2);
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  margin-left: 8px;
  font-size: 14px;
}

.search-button {
  background-color: #007aff;
  color: #fff;
  border-radius: 4px;
  padding: 0 15px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  border: none;
}

/* 内容区域 */
.content-container {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.table-container {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  overflow: hidden;
}

.table-header {
  background-color: #f7f8fa;
}

.table-row:hover {
  background-color: #f5f9ff !important;
}

.task-id {
  font-family: monospace;
  color: #555;
  font-weight: 500;
}

.date-cell {
  color: #555;
  font-size: 14px;
}

.template-name {
  color: #333;
  font-size: 14px;
  font-weight: 500;
  padding: 4px 8px;
  background-color: #f5f5f5;
  border-radius: 4px;
  display: inline-block;
}

/* 图片网格 */
.image-preview-container {
  padding: 8px;
}

.image-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}

.image-item {
  width: 60px;
  height: 60px;
  border-radius: 6px;
  overflow: hidden;
  border: 1px solid #eee;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  transition: transform 0.2s;
}

.image-item:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.more-images {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
  border-radius: 6px;
  color: #666;
  font-size: 14px;
  font-weight: 500;
}

.no-image-text {
  color: #999;
  font-size: 14px;
  font-style: italic;
}

/* 操作按钮样式优化 */
.action-buttons-wrapper {
  padding: 5px;
}

.action-buttons {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
}

.action-btn {
  min-width: 80px;
  height: 34px;
  border: none;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 12px;
  font-size: 13px;
  font-weight: 500;
  gap: 6px;
  position: relative;
  overflow: hidden;
  color: white;
  transition: all 0.25s ease;
}

.action-btn::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(255, 255, 255, 0);
  transition: background-color 0.2s;
}

.action-btn:active::after {
  background-color: rgba(255, 255, 255, 0.2);
}

.btn-hover {
  transform: translateY(1px);
  box-shadow: none !important;
}

.print-btn {
  background: linear-gradient(135deg, #1a73e8, #0d62d1);
  box-shadow: 0 2px 4px rgba(26, 115, 232, 0.3);
}

.download-btn {
  background: linear-gradient(135deg, #34a853, #2a8a42);
  box-shadow: 0 2px 4px rgba(52, 168, 83, 0.3);
}

.delete-btn {
  background: linear-gradient(135deg, #ea4335, #d33426);
  box-shadow: 0 2px 4px rgba(234, 67, 53, 0.3);
}

/* 图片预览弹窗 */
.preview-popup {
  width: 90vw;
  max-width: 900px;
  height: 80vh;
  background-color: #fff;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 5px 25px rgba(0,0,0,0.2);
}

.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #eee;
}

.preview-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.close-button {
  background: transparent;
  border: none;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.close-button:hover {
  background-color: #f5f5f5;
}

.preview-swiper {
  flex: 1;
  width: 100%;
  background-color: #222;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.preview-footer {
  padding: 12px 20px;
  display: flex;
  justify-content: center;
  border-top: 1px solid #eee;
}

.preview-counter {
  color: #666;
  font-size: 14px;
}

/* 加载、错误和空状态 */
.loading-container, .error-container, .empty-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 0;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.loader {
  width: 48px;
  height: 48px;
  border: 4px solid #007aff;
  border-bottom-color: transparent;
  border-radius: 50%;
  animation: rotation 1s linear infinite;
  margin-bottom: 20px;
}

@keyframes rotation {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text, .error-text, .empty-text {
  margin-top: 15px;
  font-size: 16px;
  color: #555;
  font-weight: 500;
}

.error-text {
  color: #f56c6c;
}

.empty-image {
  width: 120px;
  height: 120px;
  opacity: 0.7;
  margin-bottom: 10px;
}

.empty-tip {
  margin-top: 10px;
  color: #999;
  font-size: 14px;
}

.retry-button {
  margin-top: 20px;
  background-color: #f56c6c;
  color: white;
  border: none;
  padding: 6px 20px;
  border-radius: 4px;
}

/* 分页控件 */
.pagination-box {
  padding: 20px;
  display: flex;
  justify-content: center;
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .page-header {
    flex-direction: column;
    align-items: flex-start;
    padding: 12px 15px;
  }
  
  .search-container {
    width: 100%;
    margin: 15px 0 0 0;
  }
  
  .search-box {
    width: 100%;
  }
  
  .image-item {
    width: 45px;
    height: 45px;
  }
  
  .more-images {
    width: 45px;
    height: 45px;
  }
  
  .action-buttons {
    flex-direction: column;
    gap: 6px;
  }
  
  .action-btn {
    width: 100%;
    min-width: auto;
    font-size: 12px;
    padding: 0 8px;
    height: 30px;
  }
}

/* 触摸设备优化 */
@media (hover: hover) {
  .print-btn:hover {
    background: linear-gradient(135deg, #0d62d1, #0951b3);
    box-shadow: 0 4px 8px rgba(26, 115, 232, 0.4);
  }
  
  .download-btn:hover {
    background: linear-gradient(135deg, #2a8a42, #207332);
    box-shadow: 0 4px 8px rgba(52, 168, 83, 0.4);
  }
  
  .delete-btn:hover {
    background: linear-gradient(135deg, #d33426, #bd2918);
    box-shadow: 0 4px 8px rgba(234, 67, 53, 0.4);
  }
}
</style>