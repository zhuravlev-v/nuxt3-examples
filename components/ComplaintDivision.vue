<template>
  <div class="complaint-division">
    <h1 :class="['complaint-division__title', `${viewport.isLessThan('XL') ? 'h4-semibold' : 'h3-semibold'}`]">
      Жалобы
    </h1>
    <alert-warning-white
      title="Столкнулись с нарушениями при получении услуги?"
      descriptionTitle="Подайте жалобу и получите ответ в течение 15 дней"
      class="complaint-alert"
    >
      <ul v-if="alert.links?.length" class="paragraph-md-medium complaint-alert__list">
        <li v-for="item of alert.links" :key="item.id" class="complaint-alert__list-item">
          <nuxt-link :to="item.to" class="link link_primary-blue complaint-alert__link">{{ item.text }}</nuxt-link>
        </li>
      </ul>
    </alert-warning-white>
    <div class="complaint-division__head">
      <h2 class="h5-semibold">История жалоб</h2>
      <template v-if="complaints[Status.all].length">
        <base-tabs
          v-model="activeDivision"
          :items="divisions"
          :is-loading="false"
          class="complaint-division__tabs"
        />
      </template>
    </div>
    <div class="complaint-division__body">
      <notification-card-list
        :is-loading="isLoading"
        :notifications="complaints[activeDivision]"
        :empty-message="bookingsEmpty"
        @card-click="onCardClick"
      />
      <pagination
        v-if="pagination.isShown"
        :page-size="pagination.params.limit"
        :current-page="pagination.currentPage"
        :total="pagination.totalItems"
        :show-more-btn="false"
        :disabled="isLoading"
        @current-changed="(page: number) => pagination.onChanged(page)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
/* types */
import type { Reactive } from 'vue';
import type { INotificationCard, INotificationCardPayload } from '~/models/NotificationCard';
import type { IComplaintItem } from '~/models/complaint/ComplaintItem';
/* components */
import Pagination from '~/components/Pagination.vue';
import BaseTabs from '~/components/BaseTabs.vue';
import AlertWarningWhite from '~/components/AlertWarningWhite.vue';
import NotificationCardList from '~/components/notification-card/NotificationCardList.vue';
/* composables */
import usePagination from '~/composables/usePagination';
import useComplaint from '~/composables/useComplaint';
/* utils */
import { mapToNotificationCardFromComplaint } from '~/utils/mapToNotificationCard';


const viewport = useViewport();

enum Status {
  all = 'all',
  open = 'open',
  closed = 'closed',
}

const divisions = ref([
  { id: Status.all, name: 'Все' },
  { id: Status.open, name: 'Активные' },
  { id: Status.closed, name: 'Закрытые' },
]);

const activeDivision = ref(Status.all);

interface IComplaints {
  [key: string]: INotificationCard[];
}

const complaints: Reactive<IComplaints> = reactive({
  [Status.all]: [],
  [Status.open]: [],
  [Status.closed]: [],
});

const bookingsEmpty = computed(() => {
  return {
    'all': 'Вы пока не подавали жалоб',
    'open': 'У вас пока нет активных жалоб',
    'closed': 'У вас пока нет закрытых жалоб',
  }[activeDivision.value];
});

const router = useRouter();

const onCardClick = ({action, notification}: INotificationCardPayload) => {
  if (action === 'self') {
    router.push(`/lk/complaints/info/${notification.id}`)
  }
};

const alert = {
  links: [
    { id: '1', to: '/lk/complaints/form/27', text: 'Жалоба на нарушения или неправомерные действия чиновников' },
    { id: '1', to: '/lk/complaints/form/28', text: 'Жалоба на решения и действия (бездействие) работника многофункционального центра' },
    { id: '1', to: '/lk/complaints/form/29', text: 'Жалоба на решения и действия (бездействие) многофункционального центра' },
  ]
};

/* api */

const complaint = useComplaint();
const pagination = usePagination();

const fetchParams = reactive({
  activeDivision
});

const isLoading = ref(false);
const skipWatchParams = ref(false);

type FetchParams = {
  limit: number;
  offset: number;
  type: keyof typeof Status;
}

const fetchData = async (params: FetchParams) => {
  try {
    isLoading.value = true;
    const data = await complaint.getList(params);
    if (data) {
      pagination.totalItems = data.total_count || 0;
      pagination.params.limit = data.limit || 0;
      pagination.params.offset = data.offset || 0;
      complaints[activeDivision.value] = data.items?.map((item: IComplaintItem) => mapToNotificationCardFromComplaint(item));
    }
  } catch (error) {
    console.log(error)
  }
  finally {
    isLoading.value = false;
  }
};

watch(fetchParams, async (newParams) => {
  console.log('watch(fetchParams')
  if (skipWatchParams.value) return;
  skipWatchParams.value = true;
  pagination.reset();
  await fetchData({  type: newParams.activeDivision, ...pagination.params });
  skipWatchParams.value = false;
}, { immediate: true });

watch(pagination.params, async (newParams) => {
  if (skipWatchParams.value) return;
  skipWatchParams.value = true;
  await fetchData({ ...newParams, type: activeDivision.value });
  skipWatchParams.value = false;
});
</script>
