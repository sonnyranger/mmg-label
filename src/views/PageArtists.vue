<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getFirestore, collection, query, orderBy, getDocs } from 'firebase/firestore'
import type { IInterview } from '@/interfaces'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
const db = getFirestore()

const interviews = ref<IInterview[]>([])
const isLoading = ref<boolean>(true)

const getAllInterviews = async <T extends IInterview>(isFilter?: boolean): Promise<T[]> => {
  let getData = query(
    collection(db, `users/${userStore.userId}/interviews`),
    orderBy('createdAt', 'desc')
  )

  const listDocs = await getDocs(getData)
  return listDocs.docs.map((doc) => {
    return doc.data() as T
  })
}

const removeDuplicates = (interviews: IInterview[]): IInterview[] => {
  const uniqueArtists = new Set()
  return interviews.filter((interview) => {
    if (uniqueArtists.has(interview.artist)) {
      return false // Пропустить дубликат
    }
    uniqueArtists.add(interview.artist)
    return true // Сохранить уникальный артист
  })
}

onMounted(async () => {
  const listInterviews: Array<IInterview> = await getAllInterviews()
  interviews.value = removeDuplicates(listInterviews) // Удаление дубликатов
  isLoading.value = false
})
</script>

<template>
  <app-progress class="flex justify-content-center" v-if="isLoading" />
  <app-message v-else-if="!isLoading && interviews.length === 0" severity="info">
    Нет артистов
  </app-message>
  <div v-else>
    <h1>Наши артисты</h1>
    <app-datatable :value="interviews" tableStyle="min-width: 50rem">
      <app-column class="flex justify-content-center" field="artist" header="Артист"></app-column>
    </app-datatable>
  </div>
</template>
