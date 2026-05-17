<template>
  <div class="search-page-container">
    <!-- 1. 顶部 Hero 区域 -->
    <section class="hero-section">
      <h1 class="hero-title">遇见美好，探索世界</h1>
      <p class="hero-subtitle">搜索景点或攻略，开启你的旅行灵感</p>

      <div class="search-bar-wrap">
        <el-input v-model="searchKey" placeholder="三亚" class="capsule-input">
          <template #prefix>
            <el-icon class="search-icon"><Search /></el-icon>
          </template>
        </el-input>
        <button class="hero-search-btn">搜索</button>
      </div>
    </section>

    <!-- 2. 主体布局区域 -->
    <div class="main-layout">
      <!-- 左侧：侧边栏 -->
      <aside class="sidebar">
        <!-- 搜索历史 -->
        <div class="side-card">
          <div class="card-header">
            <span class="title">搜索历史</span>
            <span class="action"
              ><el-icon><Delete /></el-icon> 清空</span
            >
          </div>
          <ul class="history-list">
            <li v-for="h in ['三亚', '九寨沟', '西湖', '张家界', '故宫', '上海迪士尼']" :key="h">
              <div class="left">
                <el-icon><Clock /></el-icon> {{ h }}
              </div>
              <el-icon class="close-x"><Close /></el-icon>
            </li>
          </ul>
        </div>

        <!-- 热门搜索 -->
        <div class="side-card mt-20">
          <div class="card-header">
            <span class="title">热门搜索</span>
          </div>
          <div class="hot-tags">
            <span
              v-for="tag in [
                '九寨沟',
                '张家界',
                '西湖',
                '三亚',
                '桂林',
                '故宫',
                '黄山',
                '丽江',
                '成都',
                '厦门',
                '大理',
                '哈尔滨'
              ]"
              :key="tag"
              class="tag-pill"
            >
              {{ tag }}
            </span>
          </div>
        </div>

        <!-- 左下角图片区域 - 使用 search.png -->
        <div class="reserved-banner-area mt-20">
          <img :src="searchImage" alt="search" class="banner-image" />
        </div>
      </aside>

      <!-- 右侧：主结果卡片 -->
      <main class="content-card">
        <!-- Tab 切换 -->
        <div class="content-header">
          <div class="tabs">
            <span
              :class="['tab-item', { active: activeTab === 'spot' }]"
              @click="activeTab = 'spot'"
              >景点</span
            >
            <span
              :class="['tab-item', { active: activeTab === 'strategy' }]"
              @click="activeTab = 'strategy'"
              >攻略</span
            >
          </div>
        </div>

        <div class="result-stats-bar">
          <span class="stats-text">为您找到 32 个相关景点</span>
          <div class="sort-dropdown">
            综合排序 <el-icon><ArrowDown /></el-icon>
          </div>
        </div>

        <!-- 景点网格 (一行四个) -->
        <div class="spot-grid">
          <div v-for="i in 8" :key="i" class="spot-item">
            <div class="spot-img">
              <img :src="`https://picsum.photos/400/300?random=${i}`" />
            </div>
            <div class="spot-body">
              <div class="title-row">
                <span class="name">三亚亚龙湾</span>
                <span class="rating"
                  ><el-icon color="#18b57d"><StarFilled /></el-icon> 4.7</span
                >
              </div>
              <div class="loc-row">
                <el-icon><Location /></el-icon> 海南 · 三亚
              </div>
              <p class="desc">碧海蓝天，细沙白浪，度假天堂。</p>
              <div class="price-row">
                <span class="price-val">￥399 <span>起</span></span>
              </div>
            </div>
          </div>
        </div>

        <!-- 分页 -->
        <div class="pagination-footer">
          <el-pagination background layout="prev, pager, next" :total="80" class="custom-pager" />
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import {
  Search,
  Delete,
  Clock,
  Close,
  StarFilled,
  Location,
  ArrowDown
} from '@element-plus/icons-vue'
// 正确导入图片资源
import searchImage from '@/assets/search.png'

const searchKey = ref('三亚')
const activeTab = ref('spot')
</script>

<style scoped>
/* 整体背景与布局 */
.search-page-container {
  min-height: 100vh;
  background: #f1f6f5;
  background-image: radial-gradient(at top right, #e0f0e9 0%, transparent 40%);
  padding-bottom: 60px;
}

/* 1. Hero Section */
.hero-section {
  text-align: center;
  padding: 80px 0 60px;
}

.hero-title {
  font-size: 34px;
  color: #1a1a1a;
  font-weight: 700;
  margin-bottom: 15px;
  letter-spacing: 2px;
}

.hero-subtitle {
  font-size: 15px;
  color: #8c93a1;
  margin-bottom: 40px;
}

.search-bar-wrap {
  display: flex;
  align-items: center;
  width: 680px;
  margin: 0 auto;
  background: #fff;
  height: 58px;
  border-radius: 30px;
  padding: 0 8px 0 25px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.04);
}

.capsule-input {
  flex: 1;
}
:deep(.capsule-input .el-input__wrapper) {
  box-shadow: none !important;
  font-size: 16px;
}

.hero-search-btn {
  background: #18b57d;
  color: #fff;
  border: none;
  height: 44px;
  padding: 0 35px;
  border-radius: 25px;
  font-weight: 600;
  cursor: pointer;
  transition: 0.3s;
}
.hero-search-btn:hover {
  background: #149b71;
}

/* 2. 主布局 */
.main-layout {
  max-width: 1240px;
  margin: 0 auto;
  display: flex;
  gap: 25px;
  padding: 0 20px;
}

/* 侧边栏 */
.sidebar {
  width: 260px;
  flex-shrink: 0;
}

.side-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.card-header .title {
  font-weight: 700;
  color: #1a1a1a;
}
.card-header .action {
  font-size: 13px;
  color: #999;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
}
.card-header .action:hover {
  color: #18b57d;
}

.history-list li {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  font-size: 14px;
  color: #5c616f;
  cursor: pointer;
  border-bottom: 1px solid #f5f7f9;
}
.history-list li:last-child {
  border-bottom: none;
}
.history-list li .left {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #8c93a1;
}
.history-list li:hover .left {
  color: #18b57d;
}
.close-x {
  color: #cbd5e0;
  font-size: 14px;
  cursor: pointer;
}
.close-x:hover {
  color: #ff6a3c;
}

.hot-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
.tag-pill {
  padding: 6px 14px;
  background: #f5f7f9;
  border-radius: 20px;
  font-size: 13px;
  color: #5c616f;
  cursor: pointer;
  transition: all 0.2s;
}
.tag-pill:hover {
  background: #e8f5f0;
  color: #18b57d;
}

/* 左下角图片区域 */
.reserved-banner-area {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
}

.banner-image {
  width: 100%;
  height: auto;
  display: block;
  object-fit: cover;
  transition: transform 0.3s;
}

.banner-image:hover {
  transform: scale(1.02);
}

/* 右侧主卡片 */
.content-card {
  flex: 1;
  background: #fff;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.02);
}

.content-header {
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 25px;
}

.tab-item {
  display: inline-block;
  padding: 0 10px 15px;
  margin-right: 40px;
  font-size: 16px;
  color: #5c616f;
  cursor: pointer;
  position: relative;
}
.tab-item.active {
  color: #18b57d;
  font-weight: 700;
}
.tab-item.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: #18b57d;
  border-radius: 2px;
}

.result-stats-bar {
  display: flex;
  justify-content: space-between;
  margin-bottom: 25px;
}
.stats-text {
  font-size: 14px;
  color: #8c93a1;
}
.sort-dropdown {
  font-size: 14px;
  color: #5c616f;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
}
.sort-dropdown:hover {
  color: #18b57d;
}

/* 景点网格 */
.spot-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}

.spot-item {
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.3s;
  cursor: pointer;
  background: #fff;
  border: 1px solid #f0f2f5;
}

.spot-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
  border-color: transparent;
}

.spot-img {
  height: 130px;
  overflow: hidden;
}
.spot-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}
.spot-item:hover .spot-img img {
  transform: scale(1.05);
}

.spot-body {
  padding: 12px 12px 16px;
}
.title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}
.title-row .name {
  font-weight: 700;
  font-size: 15px;
  color: #1a1a1a;
}
.title-row .rating {
  font-size: 13px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 3px;
  color: #5c616f;
}

.loc-row {
  font-size: 12px;
  color: #8c93a1;
  margin-bottom: 8px;
  display: flex;
  align-items: center;
  gap: 4px;
}
.desc {
  font-size: 12px;
  color: #8c93a1;
  line-height: 1.5;
  margin-bottom: 12px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.price-row {
  text-align: right;
}
.price-val {
  color: #ff6a3c;
  font-size: 18px;
  font-weight: 700;
}
.price-val span {
  font-size: 12px;
  color: #8c93a1;
  font-weight: 400;
}

.pagination-footer {
  margin-top: 40px;
  display: flex;
  justify-content: center;
}

/* 分页颜色定制 */
:deep(.custom-pager.is-background .el-pager li.is-active) {
  background-color: #18b57d !important;
}

.mt-20 {
  margin-top: 20px;
}

/* 响应式 */
@media (max-width: 1100px) {
  .spot-grid {
    grid-template-columns: repeat(3, 1fr);
  }
  .search-bar-wrap {
    width: 90%;
  }
}

@media (max-width: 900px) {
  .main-layout {
    flex-direction: column;
  }
  .sidebar {
    width: 100%;
  }
  .reserved-banner-area {
    max-width: 300px;
  }
  .spot-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 600px) {
  .spot-grid {
    grid-template-columns: 1fr;
  }
  .hero-section {
    padding: 50px 0 40px;
  }
  .hero-title {
    font-size: 26px;
  }
  .content-card {
    padding: 20px;
  }
  .tab-item {
    margin-right: 25px;
  }
}
</style>
