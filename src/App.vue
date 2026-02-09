<template>
  <div class="poster-tool-container">
    <h1 class="app-title">海报小工具</h1>
    
    <!-- 海报编辑区域 -->
    <div class="poster-editor">
      <!-- 控制区域 -->
      <div class="control-panel">
        <!-- 日期设置 -->
        <div class="control-section">
          <h3>签单日期设置</h3>
          <el-date-picker
            v-model="signDate"
            type="date"
            placeholder="选择日期"
            format="YYYY年MM月DD日"
            value-format="YYYY年MM月DD日"
            style="width: 200px"
          />
          <el-button type="primary" @click="resetDate">重置为当天</el-button>
        </div>
        
        <!-- 排名数据管理 -->
        <div class="control-section">
          <h3>排名数据管理</h3>
          <el-form :model="rankForm" class="rank-form">
            <el-form-item label="区域" prop="region">
              <el-input v-model="rankForm.region" placeholder="请输入区域" />
            </el-form-item>
            <el-form-item label="负责人" prop="person">
              <el-input v-model="rankForm.person" placeholder="请输入负责人" />
            </el-form-item>
            <el-form-item label="收入" prop="income">
              <el-input-number v-model="rankForm.income" :min="0" placeholder="请输入收入" />
            </el-form-item>
            <el-form-item label="合作品牌数" prop="brandCount">
              <el-input-number v-model="rankForm.brandCount" :min="0" placeholder="请输入合作品牌数" />
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="addRankData">添加排名数据</el-button>
              <el-button type="danger" @click="clearRankData" v-if="rankData.length > 0">清空排名数据</el-button>
            </el-form-item>
          </el-form>
        </div>
        
        <!-- 商家图片上传 -->
        <div class="control-section">
          <h3>商家图片管理</h3>
          <el-upload
            v-model:file-list="brandImages"
            multiple
            action="#"
            :auto-upload="false"
            :on-change="handleBrandImageChange"
            :before-remove="handleBrandImageRemove"
            :limit="12"
            :on-exceed="handleBrandImageExceed"
            list-type="picture-card"
          >
            <el-icon><i-ep-plus /></el-icon>
            <div class="el-upload__text">上传图片</div>
          </el-upload>
          <el-button type="danger" @click="clearBrandImages" v-if="brandImages.length > 0">清空商家图片</el-button>
        </div>
        
        <!-- 合作商家管理 -->
        <div class="control-section">
          <h3>合作商家管理</h3>
          <el-form :model="coopForm" class="coop-form">
            <el-form-item label="商家名称" prop="coopName">
              <el-input v-model="coopForm.coopName" placeholder="请输入商家名称" />
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="addCoopBrand">添加合作商家</el-button>
              <el-button type="danger" @click="clearCoopBrands" v-if="coopBrands.length > 0">清空合作商家</el-button>
            </el-form-item>
            <el-form-item>
              <el-button type="info" @click="addDefaultCoopBrands">添加默认合作商家</el-button>
            </el-form-item>
          </el-form>
        </div>
        
        <!-- 背景图上传 -->
        <div class="control-section">
          <h3>背景图管理</h3>
          <el-upload
            v-model:file-list="bgImage"
            action="#"
            :auto-upload="false"
            :on-change="handleBgImageChange"
            :before-remove="handleBgImageRemove"
            :limit="1"
            :on-exceed="handleBgImageExceed"
            list-type="picture"
          >
            <el-icon><i-ep-plus /></el-icon>
            <div class="el-upload__text">上传背景图</div>
          </el-upload>
          <el-button type="danger" @click="clearBgImage" v-if="bgImage.length > 0">清空背景图</el-button>
        </div>
        
        <!-- 顶部Logo上传 -->
        <div class="control-section">
          <h3>顶部Logo管理</h3>
          <el-upload
            v-model:file-list="topLogo"
            action="#"
            :auto-upload="false"
            :on-change="handleTopLogoChange"
            :before-remove="handleTopLogoRemove"
            :limit="1"
            :on-exceed="handleTopLogoExceed"
            list-type="picture"
          >
            <el-icon><i-ep-plus /></el-icon>
            <div class="el-upload__text">上传顶部Logo</div>
          </el-upload>
          <el-button type="danger" @click="clearTopLogo" v-if="topLogo.length > 0">清空顶部Logo</el-button>
        </div>
        
        <!-- 底部Logo上传 -->
        <div class="control-section">
          <h3>底部Logo管理</h3>
          <el-upload
            v-model:file-list="bottomLogo"
            action="#"
            :auto-upload="false"
            :on-change="handleBottomLogoChange"
            :before-remove="handleBottomLogoRemove"
            :limit="1"
            :on-exceed="handleBottomLogoExceed"
            list-type="picture"
          >
            <el-icon><i-ep-plus /></el-icon>
            <div class="el-upload__text">上传底部Logo</div>
          </el-upload>
          <el-button type="danger" @click="clearBottomLogo" v-if="bottomLogo.length > 0">清空底部Logo</el-button>
        </div>
        
        <!-- 操作按钮 -->
        <div class="control-section">
          <el-button type="success" @click="generatePoster" class="generate-btn">生成海报</el-button>
          <el-button type="info" @click="previewPoster">预览海报</el-button>
        </div>
      </div>
      
      <!-- 海报预览区域 -->
      <div class="poster-preview">
        <div class="poster-container" ref="posterRef" :style="bgImageStyle">
          <!-- 海报头部 -->
          <div class="poster-header">
            <h1 class="poster-title">中部BD品牌&连锁客户新签荣誉榜</h1>
            <div class="poster-date">截止日期：{{ signDate }}</div>
            <div class="top-logo" v-if="topLogo.length > 0">
              <img :src="topLogo[0].url" alt="top-logo" class="top-logo-img" />
            </div>
          </div>
          
          <!-- 海报主体 -->
          <div class="poster-body">
            <!-- 左边排名区域 -->
            <div class="rank-section">
              <h2 class="section-title">销售排行榜</h2>
              <div class="rank-table">
                <div class="rank-header">
                  <div class="rank-col">区域</div>
                  <div class="rank-col">负责人</div>
                  <div class="rank-col">收入</div>
                  <div class="rank-col">合作品牌数</div>
                </div>
                <div 
                  v-for="(item, index) in rankData" 
                  :key="index"
                  class="rank-row"
                >
                  <div class="rank-col">{{ item.region }}</div>
                  <div class="rank-col">{{ item.person }}</div>
                  <div class="rank-col">{{ item.income }}</div>
                  <div class="rank-col">{{ item.brandCount }}</div>
                </div>
                <div v-if="rankData.length === 0" class="rank-empty">暂无排名数据</div>
              </div>
            </div>
            
            <!-- 右边品牌区域 -->
            <div class="brand-section">
              <h2 class="section-title">合作品牌名单</h2>
              <div class="brand-grid">
                <div 
                  v-for="(image, index) in brandImages" 
                  :key="index"
                  class="brand-item"
                >
                  <img :src="image.url" alt="brand" class="brand-logo" />
                </div>
                <div v-if="brandImages.length === 0" class="brand-empty">暂无品牌图片</div>
              </div>
            </div>
          </div>
          
          <!-- 海报底部 -->
          <div class="poster-footer">
            <div class="coop-brands">
              <div 
                v-for="(brand, index) in coopBrands" 
                :key="index"
                class="coop-brand"
              >
                {{ brand }}
              </div>
              <div v-if="coopBrands.length === 0" class="coop-empty">暂无合作商家</div>
            </div>
            <div class="poster-logo">
              <img v-if="bottomLogo.length > 0" :src="bottomLogo[0].url" alt="bottom-logo" class="bottom-logo-img" />
              <span v-else>顺丰同城</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import { Plus } from '@element-plus/icons-vue'
import type { UploadFile } from 'element-plus'

// 签单日期
const signDate = ref('')

// 排名数据
const rankData = ref<any[]>([])
const rankForm = reactive({
  region: '',
  person: '',
  income: 0,
  brandCount: 0
})

// 品牌图片
const brandImages = ref<UploadFile[]>([])

// 背景图片
const bgImage = ref<UploadFile[]>([])

// 顶部Logo
const topLogo = ref<UploadFile[]>([])

// 底部Logo
const bottomLogo = ref<UploadFile[]>([])

// 合作商家
const coopBrands = ref<string[]>([])
const coopForm = reactive({
  coopName: ''
})

// 背景图样式
const bgImageStyle = computed(() => {
  if (bgImage.value.length > 0 && bgImage.value[0].url) {
    return {
      background: `url(${bgImage.value[0].url}) center/cover no-repeat`,
      backgroundColor: '#c8102e'
    }
  }
  return {
    background: 'linear-gradient(135deg, #c8102e 0%, #8b0000 100%)'
  }
})

// 海报容器引用
const posterRef = ref<HTMLElement>()

// 初始化日期为当天
onMounted(() => {
  resetDate()
})

// 重置日期为当天
const resetDate = () => {
  const today = new Date()
  const year = today.getFullYear()
  const month = String(today.getMonth() + 1).padStart(2, '0')
  const day = String(today.getDate()).padStart(2, '0')
  signDate.value = `${year}年${month}月${day}日`
}

// 添加排名数据
const addRankData = () => {
  if (!rankForm.region || !rankForm.person) {
    alert('请填写区域和负责人')
    return
  }
  rankData.value.push({ ...rankForm })
  // 重置表单
  Object.assign(rankForm, {
    region: '',
    person: '',
    income: 0,
    brandCount: 0
  })
}

// 清空排名数据
const clearRankData = () => {
  rankData.value = []
}

// 处理品牌图片上传
const handleBrandImageChange = (file: UploadFile, fileList: UploadFile[]) => {
  brandImages.value = fileList
}

// 处理品牌图片删除
const handleBrandImageRemove = (file: UploadFile, fileList: UploadFile[]) => {
  brandImages.value = fileList
  return true
}

// 处理品牌图片超出限制
const handleBrandImageExceed = () => {
  alert('最多上传12张品牌图片')
}

// 添加合作商家
const addCoopBrand = () => {
  if (!coopForm.coopName) {
    alert('请填写商家名称')
    return
  }
  coopBrands.value.push(coopForm.coopName)
  // 重置表单
  coopForm.coopName = ''
}

// 清空合作商家
const clearCoopBrands = () => {
  coopBrands.value = []
}

// 生成海报
const generatePoster = () => {
  alert('海报生成功能开发中...')
}

// 预览海报
const previewPoster = () => {
  alert('海报预览功能开发中...')
}

// 添加默认合作商家
const addDefaultCoopBrands = () => {
  const defaultBrands = ['楚湘豫甲，中原称霸，赢战214，中部牛，牛牛牛']
  for (const brand of defaultBrands) {
    if (!coopBrands.value.includes(brand)) {
      coopBrands.value.push(brand)
    }
  }
}

// 处理背景图上传
const handleBgImageChange = (file: UploadFile, fileList: UploadFile[]) => {
  bgImage.value = fileList
}

// 处理背景图删除
const handleBgImageRemove = (file: UploadFile, fileList: UploadFile[]) => {
  bgImage.value = fileList
  return true
}

// 处理背景图超出限制
const handleBgImageExceed = () => {
  alert('最多上传1张背景图')
}

// 清空背景图
const clearBgImage = () => {
  bgImage.value = []
}

// 处理顶部Logo上传
const handleTopLogoChange = (file: UploadFile, fileList: UploadFile[]) => {
  topLogo.value = fileList
}

// 处理顶部Logo删除
const handleTopLogoRemove = (file: UploadFile, fileList: UploadFile[]) => {
  topLogo.value = fileList
  return true
}

// 处理顶部Logo超出限制
const handleTopLogoExceed = () => {
  alert('最多上传1张顶部Logo图片')
}

// 清空顶部Logo
const clearTopLogo = () => {
  topLogo.value = []
}

// 处理底部Logo上传
const handleBottomLogoChange = (file: UploadFile, fileList: UploadFile[]) => {
  bottomLogo.value = fileList
}

// 处理底部Logo删除
const handleBottomLogoRemove = (file: UploadFile, fileList: UploadFile[]) => {
  bottomLogo.value = fileList
  return true
}

// 处理底部Logo超出限制
const handleBottomLogoExceed = () => {
  alert('最多上传1张底部Logo图片')
}

// 清空底部Logo
const clearBottomLogo = () => {
  bottomLogo.value = []
}
</script>

<style scoped>
.poster-tool-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.app-title {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.poster-editor {
  display: flex;
  gap: 20px;
}

.control-panel {
  flex: 1;
  background: #f5f7fa;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.08);
}

.control-section {
  margin-bottom: 20px;
  padding: 15px;
  background: #fff;
  border-radius: 6px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
}

.control-section h3 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #303133;
  font-size: 16px;
  font-weight: 600;
}

.rank-form, .coop-form {
  margin-bottom: 15px;
}

.poster-preview {
  flex: 2;
}

.poster-container {
  width: 100%;
  background: linear-gradient(135deg, #c8102e 0%, #8b0000 100%);
  color: #fff;
  padding: 40px;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  position: relative;
  overflow: hidden;
}

.poster-header {
  text-align: center;
  margin-bottom: 40px;
  position: relative;
}

.poster-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.poster-date {
  position: absolute;
  top: 40px;
  right: 120px;
  font-size: 14px;
  opacity: 0.9;
}

/* 顶部Logo样式 */
.top-logo {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

.top-logo-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 50%;
}

/* 底部Logo样式 */
.bottom-logo-img {
  max-width: 120px;
  max-height: 40px;
  object-fit: contain;
  border-radius: 4px;
}

.poster-body {
  display: flex;
  gap: 40px;
  margin-bottom: 40px;
}

.rank-section, .brand-section {
  flex: 1;
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
  padding-bottom: 10px;
}

.rank-table {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  overflow: hidden;
}

.rank-header {
  display: flex;
  background: rgba(255, 255, 255, 0.2);
  padding: 12px;
  font-weight: bold;
}

.rank-row {
  display: flex;
  padding: 10px 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.rank-row:nth-child(even) {
  background: rgba(255, 255, 255, 0.05);
}

.rank-col {
  flex: 1;
  text-align: center;
}

.rank-empty, .brand-empty, .coop-empty {
  padding: 40px;
  text-align: center;
  opacity: 0.7;
  font-style: italic;
}

.brand-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.brand-item {
  background: rgba(255, 255, 255, 0.1);
  padding: 15px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100px;
}

.brand-logo {
  max-width: 100%;
  max-height: 80px;
  object-fit: contain;
}

.poster-footer {
  text-align: center;
  margin-top: 40px;
  border-top: 2px solid rgba(255, 255, 255, 0.3);
  padding-top: 30px;
}

.coop-brands {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  margin-bottom: 20px;
}

.coop-brand {
  background: rgba(255, 255, 255, 0.1);
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
}

.poster-slogan {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 20px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.poster-logo {
  font-size: 18px;
  font-weight: bold;
  opacity: 0.9;
}

.generate-btn {
  width: 100%;
  margin-top: 10px;
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .poster-editor {
    flex-direction: column;
  }
  
  .poster-body {
    flex-direction: column;
    gap: 30px;
  }
  
  .brand-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .poster-container {
    padding: 20px;
  }
  
  .poster-title {
    font-size: 20px;
  }
  
  .brand-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .poster-title {
    font-size: 16px;
  }
  
  .section-title {
    font-size: 14px;
  }
  
  .rank-header, .rank-row {
    font-size: 12px;
    padding: 8px;
  }
  
  .brand-grid {
    grid-template-columns: 1fr;
  }
}
</style>
