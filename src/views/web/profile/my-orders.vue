<!-- views/web/profile/my-orders.vue -->
<template>
  <div class="my-orders">
    <div class="orders-card">
      <h2 class="card-title">我的订单</h2>

      <!-- 订单统计卡片 -->
      <div class="stats-row">
        <div class="stat-card" v-for="stat in stats" :key="stat.key" @click="activeTab = stat.key">
          <span class="stat-num">{{ stat.count }}</span>
          <span class="stat-label">{{ stat.label }}</span>
        </div>
      </div>

      <!-- 订单状态 Tab -->
      <div class="order-tabs">
        <button
          v-for="tab in tabs"
          :key="tab.key"
          :class="['tab', { active: activeTab === tab.key }]"
          @click="activeTab = tab.key"
        >
          {{ tab.label }}
          <span class="tab-count" v-if="tab.count > 0">{{ tab.count }}</span>
        </button>
      </div>

      <!-- 订单列表（滚动区域） -->
      <div class="order-list-wrapper">
        <div class="order-list">
          <div v-for="order in filteredOrders" :key="order.id" class="order-item">
            <div class="order-header">
              <div class="order-left">
                <span class="order-no">订单号：{{ order.orderNo }}</span>
                <span class="order-time">{{ order.createTime }}</span>
              </div>
              <span :class="['order-status', getStatusClass(order.status)]">{{
                order.statusText
              }}</span>
            </div>

            <div class="order-content">
              <img :src="order.image" class="order-img" />
              <div class="order-info">
                <h4>{{ order.name }}</h4>
                <div class="order-meta">
                  <span>游玩日期：{{ order.date }}</span>
                  <span>数量：{{ order.quantity }} 张</span>
                  <span>票型：{{ order.ticketType }}</span>
                </div>
                <div class="countdown-timer" v-if="order.status === 'pending'">
                  <el-icon><Timer /></el-icon>
                  <span>剩余支付时间：{{ order.countdownText || '15:00' }}</span>
                </div>
                <div class="countdown-timer days" v-if="order.status === 'unused'">
                  <el-icon><Calendar /></el-icon>
                  <span>距离游玩还有 {{ order.daysUntilTravel }} 天</span>
                </div>
              </div>
              <div class="order-price">
                <span class="total-price">¥{{ order.total }}</span>
              </div>
            </div>

            <div class="order-footer">
              <div class="order-actions">
                <button
                  v-if="order.status === 'pending'"
                  class="btn-outline"
                  @click.stop="cancelOrder(order.id)"
                >
                  取消订单
                </button>
                <button
                  v-if="order.status === 'pending'"
                  class="btn-primary"
                  @click.stop="openPayModal(order)"
                >
                  立即支付
                </button>
                <button
                  v-if="order.status === 'unused'"
                  class="btn-outline"
                  @click.stop="contactService"
                >
                  联系客服
                </button>
                <button
                  v-if="order.status === 'unused'"
                  class="btn-outline"
                  @click.stop="openDetailModal(order)"
                >
                  查看详情
                </button>
                <button
                  v-if="order.status === 'completed'"
                  class="btn-outline"
                  @click.stop="openDetailModal(order)"
                >
                  查看详情
                </button>
                <button
                  v-if="order.status === 'completed'"
                  class="btn-outline"
                  @click.stop="goReview(order.id)"
                >
                  写评价
                </button>
              </div>
            </div>
          </div>

          <!-- 空状态 -->
          <div v-if="filteredOrders.length === 0" class="empty-state">
            <div class="empty-icon">📦</div>
            <p>
              暂无{{ activeTab === 'all' ? '' : tabs.find((t) => t.key === activeTab)?.label }}订单
            </p>
            <router-link to="/category" class="go-shop-btn">去逛逛</router-link>
          </div>
        </div>
      </div>
    </div>

    <!-- 支付弹窗 - 优化版 -->
    <el-dialog
      v-model="showPayModal"
      title="订单支付"
      width="480px"
      class="payment-dialog"
      :before-close="closePayModal"
    >
      <div class="pay-form">
        <!-- 订单信息卡片 -->
        <div class="pay-order-card">
          <div class="card-icon">
            <el-icon><Ticket /></el-icon>
          </div>
          <div class="pay-order-info">
            <div class="info-group">
              <div class="info-row">
                <span class="label">订单号</span>
                <span class="value">{{ payOrder.orderNo }}</span>
              </div>
              <div class="info-row">
                <span class="label">商品名称</span>
                <span class="value">{{ payOrder.name }}</span>
              </div>
              <div class="info-row">
                <span class="label">票型</span>
                <span class="value">{{ payOrder.ticketType }}</span>
              </div>
              <div class="info-row">
                <span class="label">购买数量</span>
                <span class="value">{{ payOrder.quantity }} 张</span>
              </div>
              <div class="info-row">
                <span class="label">游玩日期</span>
                <span class="value">{{ payOrder.date }}</span>
              </div>
              <div class="info-row">
                <span class="label">联系人</span>
                <span class="value">{{ payOrder.contactName || '待填写' }}</span>
              </div>
              <div class="info-row">
                <span class="label">联系电话</span>
                <span class="value">{{ payOrder.contactPhone || '待填写' }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 金额区域 -->
        <div class="amount-section">
          <div class="amount-label">支付金额</div>
          <div class="amount-value">¥{{ payOrder.total }}</div>
        </div>

        <!-- 支付按钮 -->
        <div class="pay-actions">
          <button class="cancel-pay-btn" @click="closePayModal">取消支付</button>
          <button class="confirm-pay-btn" @click="handlePayConfirm">确认支付</button>
        </div>
      </div>
    </el-dialog>

    <!-- 订单详情弹窗 - 优化版（去掉去使用按钮） -->
    <el-dialog
      v-model="showDetailModal"
      title="订单详情"
      width="480px"
      class="detail-dialog"
      :before-close="closeDetailModal"
    >
      <div class="detail-form">
        <!-- 订单状态横幅 -->
        <div class="status-banner" :class="getStatusClass(detailOrder.status)">
          <el-icon><InfoFilled /></el-icon>
          <span>{{ detailOrder.statusText }}</span>
        </div>

        <!-- 订单信息卡片 -->
        <div class="detail-order-card">
          <div class="card-icon">
            <el-icon><Document /></el-icon>
          </div>
          <div class="detail-order-info">
            <div class="info-group">
              <div class="info-row">
                <span class="label">订单号</span>
                <span class="value">{{ detailOrder.orderNo }}</span>
              </div>
              <div class="info-row">
                <span class="label">商品名称</span>
                <span class="value">{{ detailOrder.name }}</span>
              </div>
              <div class="info-row">
                <span class="label">票型</span>
                <span class="value">{{ detailOrder.ticketType }}</span>
              </div>
              <div class="info-row">
                <span class="label">购买数量</span>
                <span class="value">{{ detailOrder.quantity }} 张</span>
              </div>
              <div class="info-row">
                <span class="label">游玩日期</span>
                <span class="value">{{ detailOrder.date }}</span>
              </div>
              <div class="info-row">
                <span class="label">下单时间</span>
                <span class="value">{{ detailOrder.createTime }}</span>
              </div>
              <div class="info-row">
                <span class="label">联系人</span>
                <span class="value">{{ detailOrder.contactName || '未填写' }}</span>
              </div>
              <div class="info-row">
                <span class="label">联系电话</span>
                <span class="value">{{ detailOrder.contactPhone || '未填写' }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 金额区域 -->
        <div class="amount-section">
          <div class="amount-label">实付金额</div>
          <div class="amount-value">¥{{ detailOrder.total }}</div>
        </div>

        <!-- 底部按钮（只保留联系客服） -->
        <div class="detail-actions">
          <button class="contact-btn" @click="contactService">
            <el-icon><Headset /></el-icon>
            联系客服
          </button>
          <button class="close-detail-btn" @click="closeDetailModal">关闭</button>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { Timer, Calendar, Ticket, Document, InfoFilled, Headset } from '@element-plus/icons-vue'

const router = useRouter()
const activeTab = ref('all')

// 弹窗状态
const showPayModal = ref(false)
const showDetailModal = ref(false)
const payOrder = ref({})
const detailOrder = ref({})

// 存储定时器ID
let timers = []

const tabs = [
  { key: 'all', label: '全部', count: 6 },
  { key: 'pending', label: '待付款', count: 2 },
  { key: 'unused', label: '待使用', count: 2 },
  { key: 'completed', label: '已完成', count: 1 },
  { key: 'cancelled', label: '已取消', count: 1 }
]

const stats = [
  { key: 'all', label: '全部订单', count: 6 },
  { key: 'pending', label: '待付款', count: 2 },
  { key: 'unused', label: '待使用', count: 2 },
  { key: 'completed', label: '已完成', count: 1 },
  { key: 'cancelled', label: '已取消', count: 1 }
]

// 计算距离游玩日期的天数
const getDaysUntilTravel = (dateStr) => {
  const today = new Date()
  today.setHours(0, 0, 0, 0)
  const travelDate = new Date(dateStr)
  travelDate.setHours(0, 0, 0, 0)
  const diffTime = travelDate - today
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  return diffDays >= 0 ? diffDays : 0
}

// 格式化倒计时
const formatCountdown = (seconds) => {
  if (seconds <= 0) return '已超时'
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

// 订单数据
const ordersData = ref([
  {
    id: 1,
    orderNo: '202501150001',
    name: '九寨沟门票',
    image: 'https://images.unsplash.com/photo-1500530855697-b586d89ba3ee?q=80&w=200&auto=format',
    date: '2025-02-01',
    quantity: 2,
    total: 598,
    createTime: '2025-01-15 14:30',
    status: 'pending',
    statusText: '待付款',
    ticketType: '成人票',
    contactName: '张三',
    contactPhone: '13800138001',
    remainingSeconds: 15 * 60
  },
  {
    id: 2,
    orderNo: '202501100002',
    name: '西湖游船票',
    image: 'https://images.unsplash.com/photo-1528127269322-539801943592?q=80&w=200&auto=format',
    date: '2025-01-20',
    quantity: 3,
    total: 297,
    createTime: '2025-01-10 10:15',
    status: 'unused',
    statusText: '待使用',
    ticketType: '成人票',
    contactName: '李四',
    contactPhone: '13800138002'
  },
  {
    id: 3,
    orderNo: '202501050003',
    name: '黄山风景区',
    image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?q=80&w=200&auto=format',
    date: '2025-01-18',
    quantity: 2,
    total: 576,
    createTime: '2025-01-05 09:20',
    status: 'pending',
    statusText: '待付款',
    ticketType: '成人票',
    contactName: '王五',
    contactPhone: '13800138003',
    remainingSeconds: 8 * 60
  },
  {
    id: 4,
    orderNo: '202412200004',
    name: '张家界套票',
    image: 'https://images.unsplash.com/photo-1501785888041-af3ef285b470?q=80&w=200&auto=format',
    date: '2024-12-25',
    quantity: 1,
    total: 488,
    createTime: '2024-12-20 09:45',
    status: 'completed',
    statusText: '已完成',
    ticketType: '成人票',
    contactName: '赵六',
    contactPhone: '13800138004'
  },
  {
    id: 5,
    orderNo: '202412150005',
    name: '故宫博物院',
    image: 'https://images.unsplash.com/photo-1588880331179-bc9b93a8cb5e?q=80&w=200&auto=format',
    date: '2025-01-20',
    quantity: 2,
    total: 120,
    createTime: '2024-12-15 16:20',
    status: 'unused',
    statusText: '待使用',
    ticketType: '儿童票',
    contactName: '小明',
    contactPhone: '13800138005'
  },
  {
    id: 6,
    orderNo: '202412010006',
    name: '漓江竹筏',
    image: 'https://images.unsplash.com/photo-1551634979-2b11f8c946fe?q=80&w=200&auto=format',
    date: '2024-12-10',
    quantity: 3,
    total: 360,
    createTime: '2024-12-01 11:00',
    status: 'cancelled',
    statusText: '已取消',
    ticketType: '成人票',
    contactName: '孙七',
    contactPhone: '13800138006'
  }
])

// 初始化
const initOrders = () => {
  ordersData.value.forEach((order) => {
    if (order.status === 'pending') {
      if (order.remainingSeconds === undefined) {
        order.remainingSeconds = 15 * 60
      }
      order.countdownText = formatCountdown(order.remainingSeconds)
    } else if (order.status === 'unused') {
      order.daysUntilTravel = getDaysUntilTravel(order.date)
    }
  })
}

// 启动倒计时
const startCountdown = () => {
  timers.forEach((timer) => clearInterval(timer))
  timers = []

  ordersData.value.forEach((order) => {
    if (order.status === 'pending' && order.remainingSeconds > 0) {
      const timer = setInterval(() => {
        if (order.remainingSeconds > 0) {
          order.remainingSeconds--
          order.countdownText = formatCountdown(order.remainingSeconds)

          if (order.remainingSeconds === 0) {
            order.status = 'cancelled'
            order.statusText = '已取消'
            order.countdownText = '已超时'
            updateStatsCount()
            clearInterval(timer)
          }
        }
      }, 1000)
      timers.push(timer)
    }
  })
}

// 更新统计
const updateStatsCount = () => {
  const pendingCount = ordersData.value.filter((o) => o.status === 'pending').length
  const unusedCount = ordersData.value.filter((o) => o.status === 'unused').length
  const completedCount = ordersData.value.filter((o) => o.status === 'completed').length
  const cancelledCount = ordersData.value.filter((o) => o.status === 'cancelled').length
  const totalCount = ordersData.value.length

  tabs.forEach((tab) => {
    if (tab.key === 'all') tab.count = totalCount
    if (tab.key === 'pending') tab.count = pendingCount
    if (tab.key === 'unused') tab.count = unusedCount
    if (tab.key === 'completed') tab.count = completedCount
    if (tab.key === 'cancelled') tab.count = cancelledCount
  })

  stats.forEach((stat) => {
    if (stat.key === 'all') stat.count = totalCount
    if (stat.key === 'pending') stat.count = pendingCount
    if (stat.key === 'unused') stat.count = unusedCount
    if (stat.key === 'completed') stat.count = completedCount
    if (stat.key === 'cancelled') stat.count = cancelledCount
  })
}

const orders = computed(() => ordersData.value)

const filteredOrders = computed(() => {
  if (activeTab.value === 'all') return orders.value
  return orders.value.filter((order) => order.status === activeTab.value)
})

const getStatusClass = (status) => {
  const map = {
    pending: 'status-pending',
    unused: 'status-unused',
    completed: 'status-completed',
    cancelled: 'status-cancelled'
  }
  return map[status] || ''
}

const openPayModal = (order) => {
  payOrder.value = { ...order }
  showPayModal.value = true
}

const closePayModal = () => {
  showPayModal.value = false
  payOrder.value = {}
}

const handlePayConfirm = () => {
  const order = ordersData.value.find((o) => o.id === payOrder.value.id)
  if (order) {
    order.status = 'unused'
    order.statusText = '待使用'
    order.daysUntilTravel = getDaysUntilTravel(order.date)
    updateStatsCount()
  }
  alert(
    `支付成功！\n订单号：${payOrder.value.orderNo}\n金额：¥${payOrder.value.total}\n我们将尽快为您出票，请留意短信通知。`
  )
  closePayModal()
}

const openDetailModal = (order) => {
  detailOrder.value = { ...order }
  showDetailModal.value = true
}

const closeDetailModal = () => {
  showDetailModal.value = false
  detailOrder.value = {}
}

const cancelOrder = (id) => {
  const order = ordersData.value.find((o) => o.id === id)
  if (order && order.status === 'pending') {
    order.status = 'cancelled'
    order.statusText = '已取消'
    updateStatsCount()
    alert('订单已取消')
  }
}

const contactService = () => {
  alert('客服热线：400-123-4567\n在线时间：09:00-21:00')
}

const goReview = (id) => {
  alert('评价功能开发中')
}

onMounted(() => {
  initOrders()
  startCountdown()
})

onUnmounted(() => {
  timers.forEach((timer) => clearInterval(timer))
})
</script>

<style scoped>
/* 我的订单 - 完整 CSS */
.my-orders {
  display: flex;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.orders-card {
  width: 900px;
  background: #fff;
  border-radius: 20px;
  padding: 32px 40px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  display: flex;
  flex-direction: column;
  max-height: calc(100vh - 140px);
}

.card-title {
  text-align: center;
  font-size: 20px;
  font-weight: 600;
  color: #1a2c3e;
  padding-bottom: 20px;
  border-bottom: 2px solid #f0f2f5;
  margin-bottom: 20px;
  flex-shrink: 0;
}

/* 统计卡片 */
.stats-row {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
  flex-shrink: 0;
}

.stat-card {
  flex: 1;
  background: #f8fafc;
  border-radius: 14px;
  padding: 10px 8px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
}

.stat-card:hover {
  background: #f1f5f9;
  transform: translateY(-2px);
}

.stat-num {
  display: block;
  font-size: 22px;
  font-weight: 700;
  color: #1a2c3e;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 12px;
  color: #8a94a6;
}

/* Tab 样式 */
.order-tabs {
  display: flex;
  gap: 4px;
  border-bottom: 1px solid #e8ecf0;
  margin-bottom: 16px;
  flex-shrink: 0;
}

.tab {
  padding: 8px 20px;
  background: none;
  border: none;
  font-size: 14px;
  font-weight: 500;
  color: #8a94a6;
  cursor: pointer;
  transition: 0.3s;
  position: relative;
  display: flex;
  align-items: center;
  gap: 6px;
}

.tab:hover {
  color: #18b57d;
}

.tab.active {
  color: #18b57d;
}

.tab.active::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  right: 0;
  height: 2px;
  background: #18b57d;
}

.tab-count {
  background: #f1f5f9;
  padding: 0 5px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: normal;
}

/* 订单列表滚动区域 */
.order-list-wrapper {
  flex: 1;
  overflow-y: auto;
  min-height: 0;
  padding-right: 4px;
}

.order-list-wrapper::-webkit-scrollbar {
  width: 6px;
}

.order-list-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

.order-list-wrapper::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.order-list-wrapper::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.order-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 订单卡片 */
.order-item {
  border: 1px solid #e8ecf0;
  border-radius: 16px;
  padding: 16px 20px;
  transition: 0.3s;
}

.order-item:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 12px;
  margin-bottom: 12px;
  border-bottom: 1px solid #f0f2f5;
}

.order-left {
  display: flex;
  gap: 20px;
  align-items: center;
}

.order-no {
  font-size: 13px;
  font-weight: 500;
  color: #4a5568;
}

.order-time {
  font-size: 12px;
  color: #8a94a6;
}

.order-status {
  font-size: 12px;
  font-weight: 500;
  padding: 4px 12px;
  border-radius: 20px;
}

.status-pending {
  background: #fff3e0;
  color: #ed8936;
}

.status-unused {
  background: #e8f8f2;
  color: #18b57d;
}

.status-completed {
  background: #e8ecf0;
  color: #8a94a6;
}

.status-cancelled {
  background: #fee;
  color: #e53e3e;
}

.order-content {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
}

.order-img {
  width: 90px;
  height: 70px;
  border-radius: 12px;
  object-fit: cover;
}

.order-info {
  flex: 1;
}

.order-info h4 {
  font-size: 15px;
  font-weight: 600;
  color: #1a2c3e;
  margin-bottom: 8px;
}

.order-meta {
  display: flex;
  gap: 20px;
  margin-bottom: 8px;
  flex-wrap: wrap;
}

.order-meta span {
  font-size: 12px;
  color: #8a94a6;
}

/* 倒计时样式 */
.countdown-timer {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #ed8936;
  background: #fff3e0;
  padding: 4px 10px;
  border-radius: 20px;
}

.countdown-timer.days {
  color: #18b57d;
  background: #e8f8f2;
}

.countdown-timer .el-icon {
  font-size: 14px;
}

.order-price {
  text-align: right;
}

.total-price {
  display: block;
  font-size: 18px;
  font-weight: 700;
  color: #ff6a00;
}

.order-footer {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding-top: 12px;
  border-top: 1px solid #f0f2f5;
}

.order-actions {
  display: flex;
  gap: 12px;
}

.btn-primary {
  padding: 6px 20px;
  background: linear-gradient(135deg, #1ec58b, #149b71);
  color: #fff;
  border: none;
  border-radius: 30px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: 0.3s;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(24, 181, 125, 0.3);
}

.btn-outline {
  padding: 6px 20px;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 30px;
  font-size: 12px;
  cursor: pointer;
  transition: 0.3s;
}

.btn-outline:hover {
  border-color: #18b57d;
  color: #18b57d;
}

.empty-state {
  text-align: center;
  padding: 60px 0;
}

.empty-icon {
  font-size: 64px;
  margin-bottom: 16px;
  opacity: 0.4;
}

.empty-state p {
  color: #8a94a6;
  margin-bottom: 20px;
}

.go-shop-btn {
  display: inline-block;
  padding: 10px 32px;
  background: linear-gradient(135deg, #1ec58b, #149b71);
  color: #fff;
  text-decoration: none;
  border-radius: 30px;
  font-size: 14px;
}

/* ==================== 支付弹窗样式 ==================== */
.payment-dialog :deep(.el-dialog) {
  border-radius: 24px !important;
  overflow: hidden;
}

.payment-dialog :deep(.el-dialog__header) {
  background: linear-gradient(135deg, #1ec58b, #149b71);
  padding: 20px 24px;
  margin: 0;
}

.payment-dialog :deep(.el-dialog__title) {
  font-size: 18px;
  font-weight: 600;
  color: #fff;
}

.payment-dialog :deep(.el-dialog__headerbtn .el-dialog__close) {
  color: #fff;
  font-size: 18px;
}

.payment-dialog :deep(.el-dialog__body) {
  padding: 24px;
}

.pay-form {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.pay-order-card {
  display: flex;
  gap: 16px;
  background: linear-gradient(135deg, #f8fafc, #fff);
  border-radius: 20px;
  padding: 20px;
  border: 1px solid #edf2f7;
}

.card-icon {
  width: 48px;
  height: 48px;
  background: linear-gradient(135deg, #1ec58b, #149b71);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-icon .el-icon {
  font-size: 24px;
  color: #fff;
}

.pay-order-info {
  flex: 1;
}

.info-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  font-size: 14px;
}

.info-row .label {
  color: #8a94a6;
  font-weight: 400;
}

.info-row .value {
  color: #1a2c3e;
  font-weight: 500;
}

.amount-section {
  background: #f8fafc;
  border-radius: 16px;
  padding: 16px 20px;
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

.amount-label {
  font-size: 14px;
  color: #8a94a6;
}

.amount-value {
  font-size: 28px;
  font-weight: 700;
  color: #ff6a00;
}

.pay-actions {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}

.cancel-pay-btn {
  flex: 1;
  padding: 14px;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 40px;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.cancel-pay-btn:hover {
  border-color: #e53e3e;
  color: #e53e3e;
}

.confirm-pay-btn {
  flex: 1;
  padding: 14px;
  background: linear-gradient(135deg, #1ec58b, #149b71);
  border: none;
  border-radius: 40px;
  font-size: 15px;
  font-weight: 600;
  color: #fff;
  cursor: pointer;
  transition: all 0.2s;
}

.confirm-pay-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(30, 197, 139, 0.3);
}

/* ==================== 详情弹窗样式 ==================== */
.detail-dialog :deep(.el-dialog) {
  border-radius: 24px !important;
  overflow: hidden;
}

.detail-dialog :deep(.el-dialog__header) {
  background: linear-gradient(135deg, #2c3e50, #1a2c3e);
  padding: 20px 24px;
  margin: 0;
}

.detail-dialog :deep(.el-dialog__title) {
  font-size: 18px;
  font-weight: 600;
  color: #fff;
}

.detail-dialog :deep(.el-dialog__headerbtn .el-dialog__close) {
  color: #fff;
  font-size: 18px;
}

.detail-dialog :deep(.el-dialog__body) {
  padding: 24px;
}

.detail-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.status-banner {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 12px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
}

.status-banner.status-pending {
  background: #fff3e0;
  color: #ed8936;
}

.status-banner.status-unused {
  background: #e8f8f2;
  color: #18b57d;
}

.status-banner.status-completed {
  background: #e8ecf0;
  color: #8a94a6;
}

.status-banner.status-cancelled {
  background: #fee;
  color: #e53e3e;
}

.detail-order-card {
  display: flex;
  gap: 16px;
  background: #f8fafc;
  border-radius: 20px;
  padding: 20px;
  border: 1px solid #edf2f7;
}

.detail-order-info {
  flex: 1;
}

.detail-actions {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}

.contact-btn {
  flex: 1;
  padding: 12px;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 40px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.contact-btn:hover {
  border-color: #1ec58b;
  color: #1ec58b;
}

.close-detail-btn {
  flex: 1;
  padding: 12px;
  background: #f5f7f9;
  border: none;
  border-radius: 40px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.close-detail-btn:hover {
  background: #e8ecf0;
}

/* 响应式 */
@media (max-width: 768px) {
  .orders-card {
    width: 95%;
    padding: 20px;
  }
  .order-meta {
    flex-direction: column;
    gap: 4px;
  }
  .order-content {
    flex-direction: column;
  }
  .order-img {
    width: 100%;
    height: 140px;
  }
  .order-footer {
    flex-direction: column;
    gap: 12px;
  }
  .order-actions {
    flex-wrap: wrap;
    justify-content: center;
  }
}
</style>
