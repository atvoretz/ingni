<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <!-- Динамически изменяем иконку и действие кнопки -->
        <q-btn
          flat
          dense
          round
          :icon="showSearch ? 'arrow_back' : 'menu'"
          aria-label="Menu"
          @click="showSearch ? toggleSearch() : toggleLeftDrawer()"
        />

        <q-toolbar-title>ingeni</q-toolbar-title>

        <div class="flex row items-center q-gutter-sm">
          <!-- Строка поиска, скрыта по умолчанию -->
          <q-input
            filled
            v-model="searchText"
            v-show="showSearch"
            dense
            debounce="300"
            placeholder="Поиск..."
            class="col"
          />
          <q-btn
            flat
            dense
            round
            icon="search"
            aria-label="Search"
            @click="toggleSearch"
          />
        </div>
      </q-toolbar>
      <q-tabs
        v-model="tab"
        @update:model-value="handleTabClick"
        dense
        no-caps
        inline-label
        class="bg-primary text-white"
        align="justify"
      >
        <q-tab
          v-for="link in essentialLinks"
          :icon="link.icon"
          :key="link.title"
          :name="link.title"
          :label="link.title"
        />
      </q-tabs>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item dense>
          <q-item-section avatar>
            <q-avatar size="50px">
              <img src="https://ingeni.app/brand/avatar.png" />
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label lines="1"
              ><strong>Андрей Тарасов</strong></q-item-label
            >
            <q-item-label caption lines="1">ИП ТАРАСОВ</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-btn icon="brightness_4" flat @click="toggleDarkMode" />
          </q-item-section>
        </q-item>

        <q-separator spaced />

        <q-item
          dense
          v-for="link in essentialLinks"
          :key="link.title"
          clickable
        >
          <q-item-section avatar>
            <q-icon :name="link.icon" />
          </q-item-section>

          <q-item-section>
            <q-item-label lines="1">{{ link.title }}</q-item-label>
            <q-item-label caption lines="1">{{ link.caption }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';

const linksList = [
  {
    title: 'Входящие запросы',
    caption: 'Журнал всех запросов',
    icon: 'text_snippet',
    link: '/',
  },
  {
    title: 'Обработанные сбои',
    caption: 'Журнал обработанных ошибок.',
    icon: 'text_snippet',
    link: '/exceptions',
  },
  {
    title: 'Ошибки сервера',
    caption: 'Что там в логах apache?',
    icon: 'storage',
    link: '/server_errors',
  },
  {
    title: 'Подключенные аккаунты',
    caption: 'Список аккаунтов amoCRM',
    icon: 'cable',
    link: '/clients',
  },
];

export default defineComponent({
  name: 'MainLayout',

  setup() {
    const leftDrawerOpen = ref(false);
    const showSearch = ref(false);
    const searchText = ref('');
    const $q = useQuasar();
    const tab = ref('');
    const essentialLinks = ref(linksList); // Объявляем как реактивное свойство
    const router = useRouter(); // Получаем доступ к экземпляру роутера

    const toggleDarkMode = () => {
      $q.dark.set(!$q.dark.isActive);
    };

    const toggleLeftDrawer = () => {
      leftDrawerOpen.value = !leftDrawerOpen.value;
    };

    const toggleSearch = () => {
      showSearch.value = !showSearch.value;
    };

    const handleTabClick = (tabTitle) => {
      const link = essentialLinks.value.find(
        (link) => link.title === tabTitle
      )?.link;
      if (link) {
        router.push(link);
      }
    };

    onMounted(() => {
      if (essentialLinks.value.length > 0) {
        tab.value = essentialLinks.value[0].title; // Теперь корректно используем .value
      }
    });

    return {
      leftDrawerOpen,
      showSearch,
      searchText,
      toggleLeftDrawer,
      toggleSearch,
      toggleDarkMode,
      tab,
      essentialLinks,
      handleTabClick,
    };
  },
});
</script>
