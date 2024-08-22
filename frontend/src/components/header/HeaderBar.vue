<template>
  <header>
    <img v-if="showLogo" :src="logoURL" />
    <Action
      v-if="showMenu"
      class="menu-button"
      icon="menu"
      :label="t('buttons.toggleSidebar')"
      @action="layoutStore.showHover('sidebar')"
    />

    <slot />

    <Action
      icon="play_arrow"
      :label="t('buttons.executeCommand')"
      @action="executeCommand"
    />

    <div
      id="dropdown"
      :class="{ active: layoutStore.currentPromptName === 'more' }"
    >
      <slot name="actions" />
    </div>

    <Action
      v-if="ifActionsSlot"
      id="more"
      icon="more_vert"
      :label="t('buttons.more')"
      @action="layoutStore.showHover('more')"
    />

    <div
      class="overlay"
      v-show="layoutStore.currentPromptName == 'more'"
      @click="layoutStore.closeHovers"
    />
  </header>
</template>

<script setup lang="ts">
import { useLayoutStore } from "@/stores/layout";
import { useFileStore } from "@/stores/file";

import { logoURL } from "@/utils/constants";

import Action from "@/components/header/Action.vue";
import { computed, useSlots } from "vue";
import { useI18n } from "vue-i18n";
import command from "@/api/commands";

defineProps<{
  showLogo?: boolean;
  showMenu?: boolean;
}>();

const layoutStore = useLayoutStore();
const slots = useSlots();

const { t } = useI18n();

const ifActionsSlot = computed(() => (slots.actions ? true : false));

const fileStore = useFileStore();

const currentPath = computed(() => {
  if (fileStore.req && fileStore.req.items) {
    const path = fileStore.req.items[fileStore.selected[0]]?.path || '/';
    return path.startsWith('/') ? path.slice(1) : path;
  }
  return '';
});

const executeCommand = () => {
  const pathString = currentPath.value;
  const commandString = "python detect.py --weight 1104mountain.pt --project outputs --name . --exist-ok --source " + pathString;

  console.log(pathString);
  console.log(commandString);
  command(
    '/',
    commandString,
    (event) => {
      // Handle incoming messages
      console.log("Received message:", event.data);
    },
    (event) => {
      // Handle WebSocket close
      console.log(pathString);
      console.log(commandString);
      console.log("WebSocket closed:", event);
    }
  );
};
</script>

<style></style>