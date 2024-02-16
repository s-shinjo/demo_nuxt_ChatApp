<template>
  <v-sheet color="line-bg" :height="sheetHeight" style="max-width: 1200px; margin: 0 auto;">
    <ChatForm ref="chatFormRef" />
  </v-sheet>
</template>

<script>
import ChatForm from "@/components/views/ChatForm.vue";

export default {
  components: {
    ChatForm,
  },
  data() {
    return {
      sheetHeight: "",
      windowHeight: "",
    };
  },
  mounted() {
    this.updateSheetHeight();
    window.addEventListener('load', this.updateSheetHeight);
    window.addEventListener('resize', this.updateSheetHeight);
    window.addEventListener('scroll', this.updateSheetHeight);
  },
  beforeDestroy() {
    window.removeEventListener('load', this.updateSheetHeight);
    window.removeEventListener('resize', this.updateSheetHeight);
    window.removeEventListener('scroll', this.updateSheetHeight);
  },
  methods: {
    updateSheetHeight() {
      // 現在のウィンドウサイズ（$nextTickより前で取得すること）
      this.windowHeight = window.innerHeight;

      this.$nextTick(() => {
        const chatFormElement = this.$refs.chatFormRef.$el;
        const chatFormHeight = chatFormElement.clientHeight;

        // リサイズ
        if (chatFormHeight < this.windowHeight * 0.9) {
          this.sheetHeight = `${this.windowHeight * 0.9}px`;
        } else {
          this.sheetHeight = "auto";
        }
      });
    },
  },

};
</script>

<style scoped></style>
