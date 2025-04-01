<template>
  <view class="app-container" :style="backgroundStyle">
    <view class="content-container">
      <view class="top-buttons">
        <view class="left-buttons">
          <view class="action-btn settings-btn" @click="navigateTo('/pages/settings/index')">
            <uni-icons type="gear" size="20" color="#1a73e8"></uni-icons>
            <text>设置</text>
          </view>
          
          <view class="action-btn exit-btn" @click="showExitModal">
            <uni-icons type="logout" size="20" color="#ea4335"></uni-icons>
            <text>退出</text>
          </view>
        </view>
        
        <view class="action-btn album-btn" @click="navigateTo('/pages/history/index')">
          <uni-icons type="images" size="20" color="#34a853"></uni-icons>
          <text>查看相册</text>
        </view>
      </view>
      
      <view class="main-button-container">
        <view class="start-photo-btn" @click="navigateTo('/pages/template/select')">
          <uni-icons type="camera" size="42" color="#1a73e8" class="mr-4"></uni-icons>
          <text>开始拍照</text>
        </view>
      </view>
    </view>
    
    <!-- 退出确认模态框 -->
    <uni-popup ref="exitModal" type="center" :mask-click="false">
      <view class="modal-container">
        <view class="modal-header">
          <view class="modal-title">确认退出</view>
          <view class="modal-close" @click="closeExitModal">
            <uni-icons type="closeempty" size="20" color="#5f6368"></uni-icons>
          </view>
        </view>
        <view class="modal-body">
          <view class="modal-icon">
            <uni-icons type="info-filled" size="36" color="#ea4335"></uni-icons>
          </view>
          <text class="modal-message">确定要退出系统吗？</text>
        </view>
        <view class="modal-footer">
          <view class="btn btn-cancel" @click="closeExitModal">取消</view>
          <view class="btn btn-confirm" @click="logout">确认退出</view>
        </view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
import uniIcons from '@/uni_modules/uni-icons/components/uni-icons/uni-icons.vue'
import uniPopup from '@/uni_modules/uni-popup/components/uni-popup/uni-popup.vue'
import { getAppBackground } from '@/uni_modules/uni-id-pages/pages/login/login-withpwd.vue';

export default {
  components: {
    uniIcons,
    uniPopup
  },
  data() {
    return {
      backgroundImage: '/static/login-bg.jpg',
      isLoggingOut: false,
    }
  },
  computed: {
    backgroundStyle() {
      console.log('页面应用背景样式:', this.backgroundImage);
      
      if (this.backgroundImage.startsWith('#')) {
        return {
          backgroundColor: this.backgroundImage,
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
  async onLoad() {
    console.log('开始页面加载，获取背景图');
    try {
      this.backgroundImage = await getAppBackground();
      console.log('成功获取背景图:', this.backgroundImage);
    } catch (e) {
      console.error('获取背景图失败:', e);
    }
  },
  methods: {
    navigateTo(url) {
      uni.navigateTo({
        url: url
      });
    },
    showExitModal() {
      this.$refs.exitModal.open();
    },
    closeExitModal() {
      this.$refs.exitModal.close();
    },
    async logout() {
      if (this.isLoggingOut) {
        return;
      }
      
      this.isLoggingOut = true;
      
      try {
        uni.showLoading({
          title: '退出中...'
        });
        
        // 无论是否能成功调用logout，都清除本地存储
        try {
          const uniIdCo = uniCloud.importObject('uni-id-co', {
            errorOptions: {
              type: 'none' // 改为none，由我们自己处理错误提示
            }
          });
          
          const result = await uniIdCo.logout();
          console.log('退出登录结果:', result);
        } catch (serviceError) {
          console.warn('服务端退出登录失败，将仅清除本地登录状态:', serviceError);
          // 即使服务端调用失败，继续执行后续操作
        }
        
        // 无论服务端是否成功，都清除本地存储
        try {
          uni.removeStorageSync('uni_id_token');
          uni.removeStorageSync('uni_id_user');
          // 清除所有可能与登录相关的缓存
          uni.removeStorageSync('uni_id_pages_userinfo');
          uni.removeStorageSync('uni-id-pages-userInfo');
          console.log('已清除本地登录数据');
        } catch (storageError) {
          console.error('清除本地存储失败:', storageError);
        }
        
        // 显示退出成功提示
        uni.showToast({
          title: '退出成功',
          icon: 'success',
          duration: 1500
        });
        
        // 跳转到登录页
        setTimeout(() => {
          uni.reLaunch({
            url: '/uni_modules/uni-id-pages/pages/login/login-withpwd'
          });
        }, 1500);
      } catch (e) {
        console.error('退出登录过程出错:', e);
        // 即使出错，也尝试重定向到登录页
        uni.showToast({
          title: '退出过程出现异常，将重新登录',
          icon: 'none',
          duration: 2000
        });
        
        setTimeout(() => {
          uni.reLaunch({
            url: '/uni_modules/uni-id-pages/pages/login/login-withpwd'
          });
        }, 2000);
      } finally {
        uni.hideLoading();
        this.isLoggingOut = false;
        this.closeExitModal();
      }
    }
  }
}
</script>

<style>
.app-container {
  position: relative;
  width: 100%;
  height: 100vh;
}

.content-container {
  position: relative;
  z-index: 2;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.top-buttons {
  display: flex;
  justify-content: space-between;
  padding: 30px 40px;
  z-index: 10;
}

.left-buttons {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.main-button-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding-bottom: 60px;
}

.start-photo-btn {
  background-color: rgba(255, 255, 255, 0.85);
  border: 3px solid #1a73e8;
  border-radius: 20px;
  padding: 36px 70px;
  font-size: 42px;
  font-weight: 700;
  color: #1a73e8;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
}

.start-photo-btn:active {
  transform: scale(1.05);
  background-color: rgba(255, 255, 255, 0.95);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.action-btn {
  background-color: rgba(255, 255, 255, 0.85);
  border-radius: 16px;
  padding: 14px 28px;
  font-size: 18px;
  font-weight: 600;
  color: #5f6368;
  display: flex;
  align-items: center;
  gap: 12px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.12);
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.action-btn:active {
  background-color: rgba(255, 255, 255, 0.95);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.18);
  transform: translateY(-2px);
}

.album-btn {
  color: #34a853;
}

.settings-btn {
  color: #1a73e8;
}

.exit-btn {
  color: #ea4335;
  border-left: 3px solid #ea4335;
}

/* 模态框样式 */
.modal-container {
  background-color: white;
  border-radius: 20px;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
  width: 90%;
  max-width: 460px;
  overflow: hidden;
  animation: modalFadeIn 0.3s ease-out;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.modal-title {
  font-size: 20px;
  font-weight: 700;
  color: #202124;
}

.modal-close {
  background: none;
  border: none;
  font-size: 20px;
  color: #5f6368;
  padding: 8px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.modal-close:active {
  background-color: #f1f3f4;
  transform: rotate(90deg);
}

.modal-body {
  padding: 30px 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.modal-icon {
  width: 80px;
  height: 80px;
  background-color: rgba(234, 67, 53, 0.1);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-message {
  font-size: 18px;
  font-weight: 500;
  color: #3c4043;
  text-align: center;
}

.modal-footer {
  display: flex;
  justify-content: center;
  gap: 16px;
  padding: 20px 24px 28px;
}

.btn {
  padding: 12px 30px;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  transition: all 0.3s ease;
  min-width: 120px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.btn-cancel {
  background-color: #f8f9fa;
  border: 1.5px solid #dadce0;
  color: #5f6368;
}

.btn-cancel:active {
  background-color: #f1f3f4;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.btn-confirm {
  background-color: #ea4335;
  border: 1.5px solid #ea4335;
  color: white;
  box-shadow: 0 4px 12px rgba(234, 67, 53, 0.25);
}

.btn-confirm:active {
  background-color: #d93025;
  box-shadow: 0 6px 16px rgba(234, 67, 53, 0.35);
  transform: translateY(-2px);
}

/* 辅助类 */
.mr-4 {
  margin-right: 16px;
}

.text-gray-700 {
  color: #3c4043;
}
</style> 