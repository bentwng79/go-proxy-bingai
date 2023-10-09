<script setup lang="ts">
import { h, ref, onMounted } from 'vue';
import { NDropdown, type DropdownOption, NModal, NInput, NInputNumber, NButton, useMessage, NImage, NForm, NFormItem, NSwitch, NTag, NSelect, NConfigProvider, lightTheme, darkTheme } from 'naive-ui';
import settingSvgUrl from '@/assets/img/setting.svg?url';
import { usePromptStore } from '@/stores/modules/prompt';
import { storeToRefs } from 'pinia';
import ChatNavItem from './ChatNavItem.vue';
import type { Component } from 'vue';
import { isMobile } from '@/utils/utils';
import CreateImage from '@/components/CreateImage/CreateImage.vue';
import { useChatStore } from '@/stores/modules/chat';
import { useUserStore } from '@/stores/modules/user';

const isShowMore = ref(false);
const isShowSettingModal = ref(false);
const isShowAdvancedSettingModal = ref(false);
const isShowSetAboutModal = ref(false);
const userToken = ref('');
const userKievRPSSecAuth = ref('');
const userRwBf = ref('');
const userMUID = ref('');
const message = useMessage();
const promptStore = usePromptStore();
const { isShowPromptSotre } = storeToRefs(promptStore);
const isShowClearCacheModal = ref(false);
const isShowCreateImageModal = ref(false);
const chatStore = useChatStore();
const { isShowChatServiceSelectModal } = storeToRefs(chatStore);
const userStore = useUserStore();
const localVersion = __APP_INFO__.version;
// const lastVersion = ref('加载中...');
const { historyEnable, themeMode, fullCookiesEnable, cookiesStr, enterpriseEnable, customChatNum, sydneyEnable, sydneyPrompt } = storeToRefs(userStore)
let cookiesEnable = ref(false);
let cookies = ref('');
let history = ref(false);
let themeModeSetting = ref('auto');
let theme = ref(lightTheme);
let settingIconStyle = ref({
  filter: 'invert(70%)',
})
const enterpriseSetting = ref(true);
const customChatNumSetting = ref(0);
const sydneySetting = ref(false);
const sydneyPromptSetting = ref('');

// const GetLastVersion = async () => {
  // const res = await fetch('https://api.github.com/repos/Harry-zklcdc/go-proxy-bingai/releases/latest');
  // const json = await res.json();
  // lastVersion.value = json.tag_name;
// };

const navType = {
  github: 'github',
  chatService: 'chatService',
  promptStore: 'promptStore',
  setting: 'setting',
  compose: 'compose',
  createImage: 'createImage',
  advancedSetting: 'advancedSetting',
  reset: 'reset',
  about: 'about',
};
const navConfigs = [
  {
    key: navType.github,
    label: 'Mr.🆖 AiSpeak',
    url: 'https://speak.mister5.net/',
  },
  // {
    // key: navType.setting,
    // label: '用戶設置',
  // },
  {
    key: navType.chatService,
    label: '伺服器設置',
  },
  {
    key: navType.promptStore,
    label: '提示詞庫',
  },
  // {
    // key: navType.compose,
    // label: '撰寫文章',
    // url: '/web/compose.html',
  // },
  // {
    // key: navType.createImage,
    // label: '圖像創建',
  // },
  {
    key: navType.advancedSetting,
    label: '進階設置',
  },
  {
    key: navType.reset,
    label: '一鍵重置',
  },
  {
    key: navType.about,
    label: '關於'
  },
];

const themeModeOptions = ref([
  {
    label: '淺色',
    value: 'light',
  }, {
    label: '深色',
    value: 'dark',
  }, {
    label: '跟隨系統',
    value: 'auto',
  }
]);

onMounted(() => {
  if (themeMode.value == 'light') {
    theme.value = lightTheme;
    settingIconStyle.value = { filter: 'invert(0%)' }
  } else if (themeMode.value == 'dark') {
    theme.value = darkTheme;
    settingIconStyle.value = { filter: 'invert(70%)' }
  } else if (themeMode.value == 'auto') {
    if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      theme.value = darkTheme;
      settingIconStyle.value = { filter: 'invert(70%)' }
    } else {
      theme.value = lightTheme;
      settingIconStyle.value = { filter: 'invert(0%)' }
    }
  }
})

const renderDropdownLabel = (option: DropdownOption) => {
  return h(ChatNavItem as Component, { navConfig: option });
};

const handleSelect = (key: string) => {
  switch (key) {
    case navType.chatService:
      {
        isShowChatServiceSelectModal.value = true;
        chatStore.checkAllSydneyConfig();
      }
      break;
    case navType.promptStore:
      {
        isShowPromptSotre.value = true;
      }
      break;
    case navType.setting:
      {
        userToken.value = userStore.getUserToken();
        userKievRPSSecAuth.value = userStore.getUserKievRPSSecAuth();
        userRwBf.value = userStore.getUserRwBf();
        userMUID.value = userStore.getUserMUID();
        history.value = historyEnable.value;
        cookiesEnable.value = fullCookiesEnable.value;
        if (cookiesEnable.value) { cookies.value = cookiesStr.value; }
        themeModeSetting.value = themeMode.value;
        isShowSettingModal.value = true;
      }
      break;
    case navType.advancedSetting:
      {
        history.value = historyEnable.value;
        themeModeSetting.value = themeMode.value;
        enterpriseSetting.value = enterpriseEnable.value;
        customChatNumSetting.value = customChatNum.value;
        sydneySetting.value = sydneyEnable.value;
        sydneyPromptSetting.value = sydneyPrompt.value;
        isShowAdvancedSettingModal.value = true;
      }
      break;
    case navType.createImage:
      {
        if (!userStore.sysConfig?.isSysCK && !userStore.getUserToken()) {
          message.warning('體驗畫圖功能需先登錄');
        }
        isShowCreateImageModal.value = true;
      }
      break;
    case navType.reset:
      {
        isShowClearCacheModal.value = true;
      }
      break;
    case navType.about:
      {
        isShowSetAboutModal.value = true;
        // GetLastVersion();
      }
      break;
    default:
      break;
  }
};
const resetCache = async () => {
  isShowClearCacheModal.value = false;
  await userStore.resetCache();
  message.success('清理完成');
  window.location.href = '/web/';
};

const saveSetting = () => {
  if (cookiesEnable.value) {
    userStore.saveCookies(cookies.value);
    cookiesStr.value = cookies.value;
  } else {
    if (!userToken.value) {
      message.warning('請先填入用戶 _U Cookie');
    } else {
      userStore.saveUserToken(userToken.value);
    }
    if (!userKievRPSSecAuth.value) {
      message.warning('請先填入用戶 KievRPSSecAuth Cookie');
    } else {
      userStore.saveUserKievRPSSecAuth(userKievRPSSecAuth.value);
    }
    if (!userRwBf.value) {
      message.warning('請先填入用戶 _RwBf Cookie');
    } else {
      userStore.saveUserRwBf(userRwBf.value);
    }
    if (!userMUID.value) {
      message.warning('請先填入用戶 MUID Cookie');
    } else {
      userStore.saveUserMUID(userMUID.value);
    }
  }
  fullCookiesEnable.value = cookiesEnable.value;
  isShowSettingModal.value = false;
};

const saveAdvancedSetting = () => {
  historyEnable.value = history.value;
  const tmpEnterpris = enterpriseEnable.value;
  enterpriseEnable.value = enterpriseSetting.value;
  customChatNum.value = customChatNumSetting.value;
  const tmpSydney = sydneyEnable.value;
  sydneyEnable.value = sydneySetting.value;
  sydneyPrompt.value = sydneyPromptSetting.value;
  if (history.value) {
    if (userStore.getUserToken()) {
      CIB.vm.sidePanel.isVisibleDesktop = true;
      document.querySelector('cib-serp')?.setAttribute('alignment', 'left');
    }
  } else {
    CIB.vm.sidePanel.isVisibleDesktop = false;
    CIB.vm.sidePanel.isVisibleMobile = false;
    document.querySelector('cib-serp')?.setAttribute('alignment', 'center');
  }
  themeMode.value = themeModeSetting.value;
  if (themeModeSetting.value == 'light') {
    CIB.changeColorScheme(0);
    theme.value = lightTheme;
    settingIconStyle.value = { filter: 'invert(0%)' }
  } else if (themeModeSetting.value == 'dark') {
    CIB.changeColorScheme(1);
    theme.value = darkTheme;
    settingIconStyle.value = { filter: 'invert(70%)' }
  } else if (themeModeSetting.value == 'auto') {
    if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      CIB.changeColorScheme(1);
      theme.value = darkTheme;
      settingIconStyle.value = { filter: 'invert(70%)' }
    } else {
      CIB.changeColorScheme(0);
      theme.value = lightTheme;
      settingIconStyle.value = { filter: 'invert(0%)' }
    }
  }
  isShowAdvancedSettingModal.value = false;
  if (tmpEnterpris != enterpriseSetting.value || tmpSydney != sydneySetting.value) {
    window.location.href = '/web/';
  }
}
</script>

<template>
  <NConfigProvider :theme="theme">
    <NDropdown v-if="isMobile()" class="select-none" :show="isShowMore" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
      <NImage class="fixed top-6 right-4 cursor-pointer z-50" :src="settingSvgUrl" alt="設置菜單" :preview-disabled="true" @click="isShowMore = !isShowMore" :style="settingIconStyle"></NImage>
    </NDropdown>
    <NDropdown v-else class="select-none" trigger="hover" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
      <NImage class="fixed top-6 right-6 cursor-pointer z-50" :src="settingSvgUrl" alt="設置菜單" :preview-disabled="true" :style="settingIconStyle"></NImage>
    </NDropdown>
    <NModal v-model:show="isShowSettingModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">用戶設置</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" require-mark-placement="right-hanging" style="margin-top: 16px;">
        <NFormItem path="cookiesEnable" label="完整 Cookie">
          <NSwitch v-model:value="cookiesEnable" :disabled="true" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="Token">
          <NInput size="large" v-model:value="userToken" type="text" placeholder="需要 _U 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="KievRPSSecAuth">
          <NInput size="large" v-model:value="userKievRPSSecAuth" type="text" placeholder="需要 KievRPSSecAuth 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="_RwBf">
          <NInput size="large" v-model:value="userRwBf" type="text" placeholder="需要 _RwBf 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="MUID">
          <NInput size="large" v-model:value="userMUID" type="text" placeholder="需要 MUID 的值" />
        </NFormItem>
        <NFormItem v-show="cookiesEnable" path="token" label="Cookies">
          <NInput size="large" v-model:value="cookies" type="text" placeholder="完整用户 Cookie" />
        </NFormItem>
      </NForm>
      <template #action>
        <NButton size="large" @click="isShowSettingModal = false">取消</NButton>
        <NButton ghost size="large" type="info" @click="saveSetting">保存</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowAdvancedSettingModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">進階設置</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" require-mark-placement="right-hanging"
        style="margin-top: 16px;">
        <NFormItem path="history" label="歷史記錄">
          <NSwitch v-model:value="history" :disabled="true" />
        </NFormItem>
        <NFormItem path="enterpriseEnable" label="企業版">
          <NSwitch v-model:value="enterpriseSetting" />
        </NFormItem>
        <NFormItem path="sydneyEnable" label="越獄模式">
          <NSwitch v-model:value="sydneySetting" :disabled="true" />
        </NFormItem>
        <NFormItem path="sydneyPrompt" label="提示詞">
          <NInput size="large" v-model:value="sydneyPromptSetting" type="text" placeholder="越獄模式提示詞" disabled />
        </NFormItem>
        <NFormItem path="themeMode" label="主題模式">
          <NSelect v-model:value="themeModeSetting" :options="themeModeOptions" size="large" placeholder="選擇主題模式" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="customChatNum" label="聊天次數">
          <NInputNumber size="large" v-model:value="customChatNumSetting" min="0" max="30" style="width: 100%;"/>
        </NFormItem>
      </NForm>
      <template #action>
        <NButton size="large" @click="isShowAdvancedSettingModal = false">取消</NButton>
        <NButton ghost size="large" type="info" @click="saveAdvancedSetting">保存</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowClearCacheModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-xl py-2">將刪除包括 Cookie 等的所有緩存？</div>
      </template>
      <template #action>
        <NButton size="large" @click="isShowClearCacheModal = false">取消</NButton>
        <NButton ghost size="large" type="error" @click="resetCache">確定</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowSetAboutModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">關於</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" size="small" style="margin-top: 16px;">
        <NFormItem path="" label="Mr.🆖 AI English Tutor">
        <NTag type="info" size="small" round>{{ 'v'+localVersion }}</NTag>
        </NFormItem>
      </NForm>
        <div class="px-2 xl:px-10">
          <p class="text-left">This application is being developed by Mr. Ng for the sake of enabling students to engage in self-directed English learning with the help of AI.</p>
          <p class="text-left">此應用程式由伍Sir開發，旨在使學生能夠在人工智能的幫助下進行自主英語學習。</p>
        </div>
      <template #action>
        <NButton ghost size="large" @click="isShowSetAboutModal = false" type="info">確定</NButton>
      </template>
    </NModal>
  <CreateImage v-model:show="isShowCreateImageModal" />
</NConfigProvider>
</template>
