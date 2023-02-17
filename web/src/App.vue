<script setup lang="ts">
import { RedoOutlined, KeyOutlined } from "@ant-design/icons-vue";
import { message } from "ant-design-vue";
import "ant-design-vue/lib/message/style/index.css";
import { useStorage } from "@vueuse/core";
import { completion, creditSummary } from "@/api";

const loading = ref(false);
const visible = ref(false);
const pickType = ref("缩减文字");
const balance = ref(0);
const pickOptions = reactive(["缩减文字", "聊天"]);

const inputValue = ref("");
const answerValue = ref("");
const api_key = useStorage("api_key", "");

const sendMessage = async () => {
  console.log(pickType.value);
  if (!inputValue.value) return;
  if (!balance.value || balance.value <= 0) {
    message.error("Please recharge!");
    return;
  }
  loading.value = true;
  let addedText = '';
  if(pickType.value === '缩减文字')
  {
    addedText = '请将下面文章缩减到五十字以内：';
  }
  const text = addedText + inputValue.value;
  const data: any = await completion(text);
  const replyMessage = data?.choices
    ? data.choices[0].text
    : data?.error?.message;
  answerValue.value = replyMessage;
  loading.value = false;
};

const onMenuChange = (menu: string) => {
  pickType.value = menu;
};

const refushCredit = async () => {
  const summary: any = await creditSummary();
  balance.value = summary.total_available;
};
const updateApiKey = () => {
  visible.value = false;
  window.location.reload();
};

onMounted(async () => {
  await refushCredit();
});
</script>

<template>
  <div id="layout">
    <header id="header" class="flex items-center bg-dark-50 text-white h-10 select-none">
      <span class="text-size-5 px-5">ChatGPT</span>

      <a-tooltip>
        <template #title>自定义API_KEY</template>
        <KeyOutlined class="pl-3 cursor-pointer !text-red-400" @click="visible = true" />
      </a-tooltip>
      <span class="absolute right-3">
        当前余额：{{ balance }}
        <a-tooltip>
          <template #title>刷新余额</template>
          <RedoOutlined @click="refushCredit" />
        </a-tooltip>
      </span>
    </header>
    <div id="layout-body">
      <main id="main" class="p-10">
        <a-dropdown :overlayStyle="{ width: '180px' }" placement="bottomLeft" :trigger="['click']">
          <a-button>{{ pickType }}</a-button>
          <template #overlay>
            <a-menu>
              <a-menu-item v-for="(menuItem, menuI) in pickOptions" :key="menuI" @click="onMenuChange(menuItem)">
                {{ menuItem }}
              </a-menu-item>
            </a-menu>
          </template>
        </a-dropdown>
        <div class="flex items-center mt-4">
          <div class="flex-1 min-w-md">
            <a-textarea v-model:value="inputValue" :auto-size="{ minRows: 10, maxRows: 10 }" placeholder="请输入..."
              class="appearance-none pl-10 py-2 w-full bg-white border border-gray-300 rounded-full text-sm placeholder-gray-800 focus:outline-none focus:border-blue-500 focus:border-blue-500 focus:shadow-outline-blue" />
          </div>
          <a-button class="mx-4" type="primary" :loading="loading" :disabled="!inputValue" @click="sendMessage">
            Go
          </a-button>
          <div class="flex-1 bg-light min-w-md">
            <a-textarea readonly :bordered="false" v-model:value="answerValue" :auto-size="{ minRows: 10, maxRows: 10 }"
              placeholder=""
              class="appearance-none pl-10 py-2 w-full bg-white border border-gray-300 rounded-full text-sm placeholder-gray-800 focus:outline-none focus:border-blue-500 focus:border-blue-500 focus:shadow-outline-blue" />
          </div>
        </div>
      </main>
    </div>
    <a-modal v-model:visible="visible" title="更新API_KEY" @ok="updateApiKey" okText="更新" cancelText="关闭">
      <a-alert message="API_KEY往往是sk-开头的字符串" type="info" />
      <div class="mt-2"></div>
      <a-input v-model:value="api_key" placeholder="请输入API_KEY" />
    </a-modal>
  </div>
</template>

<style scoped>
body,
html {
  margin: 0;
  padding: 0;
}

#layout {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  background-color: #f0f2f5;
}

#header {
  box-shadow: 2px 5px 5px 0px rgba(102, 102, 102, 0.5);
  flex-shrink: 0;
}

#layout-body {
  flex-grow: 2;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

#footer {
  height: 100px;
  flex-shrink: 0;
}

#main {
  flex-grow: 1;
}
</style>
