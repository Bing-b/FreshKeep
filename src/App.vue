<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import {
  AlertTriangle,
  CalendarClock,
  CheckCircle2,
  Clock3,
  Bell,
  BellRing,
  Plus,
  Refrigerator,
  Search,
  Trash2,
} from 'lucide-vue-next'

const storageKey = 'freshkeep.foods.v1'
const notifiedKey = 'freshkeep.notified.v1'

const todayIso = () => {
  const today = new Date()
  today.setHours(0, 0, 0, 0)
  return today.toISOString().slice(0, 10)
}

const uid = () => crypto.randomUUID?.() ?? `${Date.now()}-${Math.random()}`

const foodName = ref('')
const expiryDate = ref(todayIso())
const remindDays = ref(3)
const query = ref('')
const statusFilter = ref('all')
const foods = ref([])
const notificationPermission = ref('Notification' in window ? Notification.permission : 'unsupported')
let reminderTimer

const todayTime = computed(() => new Date(todayIso()).getTime())

const enrichedFoods = computed(() =>
  foods.value
    .map((food) => {
      const diffDays = Math.ceil((new Date(food.expiryDate).getTime() - todayTime.value) / 86400000)
      let status = 'fresh'
      if (diffDays < 0) status = 'expired'
      else if (diffDays <= food.remindDays) status = 'soon'

      return {
        ...food,
        diffDays,
        status,
      }
    })
    .sort((a, b) => new Date(a.expiryDate) - new Date(b.expiryDate))
)

const filteredFoods = computed(() => {
  const keyword = query.value.trim().toLowerCase()

  return enrichedFoods.value.filter((food) => {
    const matchesKeyword = !keyword || food.name.toLowerCase().includes(keyword)
    const matchesStatus = statusFilter.value === 'all' || food.status === statusFilter.value
    return matchesKeyword && matchesStatus
  })
})

const stats = computed(() => ({
  total: foods.value.length,
  soon: enrichedFoods.value.filter((food) => food.status === 'soon').length,
  expired: enrichedFoods.value.filter((food) => food.status === 'expired').length,
  fresh: enrichedFoods.value.filter((food) => food.status === 'fresh').length,
}))

const alertFoods = computed(() => enrichedFoods.value.filter((food) => food.status !== 'fresh'))
const canNotify = computed(() => notificationPermission.value === 'granted')

const notificationText = computed(() => {
  if (notificationPermission.value === 'unsupported') return '当前浏览器不支持系统通知'
  if (notificationPermission.value === 'granted') return '通知已开启'
  if (notificationPermission.value === 'denied') return '通知被关闭，请在浏览器或系统设置中允许'
  return '开启到期弹窗提醒'
})

const addFood = () => {
  const name = foodName.value.trim()
  if (!name || !expiryDate.value) return

  foods.value.push({
    id: uid(),
    name,
    expiryDate: expiryDate.value,
    remindDays: Number(remindDays.value),
    createdAt: new Date().toISOString(),
  })

  foodName.value = ''
  expiryDate.value = todayIso()
  remindDays.value = 3
}

const removeFood = (id) => {
  foods.value = foods.value.filter((food) => food.id !== id)
}

const statusText = (food) => {
  if (food.status === 'expired') return `已过期 ${Math.abs(food.diffDays)} 天`
  if (food.diffDays === 0) return '今天到期'
  if (food.status === 'soon') return `${food.diffDays} 天后到期`
  return `还有 ${food.diffDays} 天`
}

const readNotified = () => {
  try {
    return JSON.parse(localStorage.getItem(notifiedKey)) ?? {}
  } catch {
    return {}
  }
}

const writeNotified = (value) => {
  localStorage.setItem(notifiedKey, JSON.stringify(value))
}

const sendFoodNotification = async (food) => {
  if (!canNotify.value) return

  const registration = await navigator.serviceWorker?.ready
  const title = food.status === 'expired' ? '食品已过期' : '食品快到期了'
  const body = `${food.name}：${statusText(food)}，过期时间 ${food.expiryDate}`
  const options = {
    body,
    icon: '/icons/icon.svg',
    badge: '/icons/icon.svg',
    tag: `freshkeep-${food.id}-${food.expiryDate}`,
    renotify: false,
  }

  if (registration?.showNotification) {
    registration.showNotification(title, options)
    return
  }

  new Notification(title, options)
}

const checkExpiryNotifications = () => {
  if (!canNotify.value) return

  const notified = readNotified()
  const today = todayIso()

  alertFoods.value.forEach((food) => {
    const notifyId = `${food.id}:${food.expiryDate}:${food.status}:${today}`
    if (notified[notifyId]) return

    sendFoodNotification(food)
    notified[notifyId] = true
  })

  writeNotified(notified)
}

const requestNotifications = async () => {
  if (!('Notification' in window)) {
    notificationPermission.value = 'unsupported'
    return
  }

  notificationPermission.value = await Notification.requestPermission()
  checkExpiryNotifications()
}

onMounted(() => {
  const saved = localStorage.getItem(storageKey)

  if (saved) {
    try {
      foods.value = JSON.parse(saved)
    } catch {
      foods.value = []
    }
  }

  checkExpiryNotifications()
  reminderTimer = window.setInterval(checkExpiryNotifications, 60 * 60 * 1000)
  document.addEventListener('visibilitychange', checkExpiryNotifications)
})

onBeforeUnmount(() => {
  window.clearInterval(reminderTimer)
  document.removeEventListener('visibilitychange', checkExpiryNotifications)
})

watch(
  foods,
  (value) => {
    localStorage.setItem(storageKey, JSON.stringify(value))
    checkExpiryNotifications()
  },
  { deep: true }
)
</script>

<template>
  <main class="app-shell">
    <section class="topbar">
      <div class="brand">
        <div class="brand-mark" aria-hidden="true">
          <Refrigerator :size="26" />
        </div>
        <div>
          <p class="eyebrow">FreshKeep</p>
          <h1>食品保质期记录</h1>
        </div>
      </div>

      <div class="summary" aria-label="食品状态汇总">
        <div>
          <span>{{ stats.total }}</span>
          <small>全部</small>
        </div>
        <div>
          <span>{{ stats.soon }}</span>
          <small>临期</small>
        </div>
        <div>
          <span>{{ stats.expired }}</span>
          <small>过期</small>
        </div>
      </div>
    </section>

    <section v-if="alertFoods.length" class="notice" role="status" aria-live="polite">
      <AlertTriangle :size="22" />
      <div>
        <strong>{{ alertFoods.length }} 件食品需要留意</strong>
        <p>{{ alertFoods[0].name }} {{ statusText(alertFoods[0]) }}</p>
      </div>
    </section>

    <section class="notify-panel" :class="{ enabled: canNotify }">
      <div class="notify-copy">
        <BellRing v-if="canNotify" :size="22" />
        <Bell v-else :size="22" />
        <div>
          <strong>{{ notificationText }}</strong>
          <p>安装为 PWA 后，应用打开或回到前台时会提醒临期和过期食品。</p>
        </div>
      </div>
      <button
        type="button"
        class="secondary-button"
        :disabled="notificationPermission === 'unsupported' || notificationPermission === 'denied'"
        @click="requestNotifications"
      >
        <Bell :size="18" />
        {{ canNotify ? '立即检查' : '允许通知' }}
      </button>
    </section>

    <section class="workspace">
      <form class="entry-panel" @submit.prevent="addFood">
        <div class="panel-heading">
          <CalendarClock :size="24" />
          <h2>新增食品</h2>
        </div>

        <label>
          食品名称
          <input v-model="foodName" type="text" placeholder="例如：牛奶、鸡蛋、草莓" autocomplete="off" />
        </label>

        <label>
          过期时间
          <input v-model="expiryDate" type="date" />
        </label>

        <label>
          提前提醒天数
          <input v-model.number="remindDays" type="number" min="0" max="365" />
        </label>

        <button class="primary-button" type="submit" :disabled="!foodName.trim() || !expiryDate">
          <Plus :size="20" />
          添加记录
        </button>
      </form>

      <section class="list-panel">
        <div class="list-tools">
          <div class="search-box">
            <Search :size="18" />
            <input v-model="query" type="search" placeholder="搜索食品" />
          </div>

          <div class="tabs" aria-label="筛选食品状态">
            <button type="button" :class="{ active: statusFilter === 'all' }" @click="statusFilter = 'all'">
              全部
            </button>
            <button type="button" :class="{ active: statusFilter === 'soon' }" @click="statusFilter = 'soon'">
              临期
            </button>
            <button type="button" :class="{ active: statusFilter === 'expired' }" @click="statusFilter = 'expired'">
              过期
            </button>
          </div>
        </div>

        <div v-if="filteredFoods.length" class="food-list">
          <article v-for="food in filteredFoods" :key="food.id" class="food-card" :class="food.status">
            <div class="food-main">
              <div class="status-icon" aria-hidden="true">
                <CheckCircle2 v-if="food.status === 'fresh'" :size="22" />
                <Clock3 v-else-if="food.status === 'soon'" :size="22" />
                <AlertTriangle v-else :size="22" />
              </div>
              <div>
                <h3>{{ food.name }}</h3>
                <p>过期时间：{{ food.expiryDate }}</p>
              </div>
            </div>

            <div class="food-meta">
              <span>{{ statusText(food) }}</span>
              <button type="button" class="icon-button" title="删除记录" @click="removeFood(food.id)">
                <Trash2 :size="18" />
              </button>
            </div>
          </article>
        </div>

        <div v-else class="empty-state">
          <Refrigerator :size="38" />
          <h2>还没有匹配的食品记录</h2>
          <p>添加食品名称和过期时间后，这里会自动显示剩余天数和提醒状态。</p>
        </div>
      </section>
    </section>
  </main>
</template>
