<script setup lang="ts">
import EventCard from '@/components/EventCard.vue'
import { type Auctions } from '@/types'
import { ref, onMounted, computed, watchEffect } from 'vue'
import EventService from '@/services/EventService'
import BaseInput from '@/components/BaseInput.vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const events = ref<Auctions[] | null>(null)
const totalEvents = ref(0)
const hasNexPage = computed(() => {
  const totalPages = Math.ceil(totalEvents.value / 3)
  return page.value < totalPages
})
const props = defineProps({
  page: {
    type: Number,
    required: true
  }
})
const page = computed(() => props.page)
onMounted(() => {
  watchEffect(() => {
    EventService.getEvents(5, page.value)
      .then((response) => {
        events.value = response.data
        totalEvents.value = response.headers['x-total-count']
        console.log(events.value)
      })
      .catch(() => {
        router.push({ name: 'network-error-view' })
      })
  })
})


const keyword = ref('')
function updateKeyword (value: string) {
  let queryFunction;
  if (keyword.value === '' ) {
    queryFunction = EventService.getEvents(5, page.value)
  } else {
    queryFunction = EventService.getEventsByKeyword(keyword.value , 5, page.value)
  }
  queryFunction.then((response) => {
    events.value = response.data
    console.log('events' ,events.value)
    totalEvents.value = response.headers['x-total-count']
    console.log('totalEvent', totalEvents)
  }).catch(() => {
    router.push({ name: 'network-error-view' })
  })
}
</script>

<template>
  <h1>Events For Good</h1>
  <!-- new element -->
  <div class="flex flex-col items-center">
    <div>
      <BaseInput v-model="keyword" label="Search..." class="w-full" @input='updateKeyword'/>
    </div>
    <EventCard v-for="event in events" :key="event.id" :auction="event" />
    <div class="pagination">
      <RouterLink
        id="page-prev"
        :to="{ name: 'event-list-view', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
        >&#60; Prev Page</RouterLink
      >

      <RouterLink
        id="page-next"
        :to="{ name: 'event-list-view', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNexPage"
        >Next Page &#62;</RouterLink
      >
    </div>
  </div>
</template>

<style scoped>

.pagination {
  display: flex;
  width: 290px;
}
.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
