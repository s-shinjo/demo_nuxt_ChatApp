<template>
  <v-container v-if="isSendingMessage">
    <v-row cols="12">
      <v-col cols="6"></v-col>
      <v-col cols="6">
        <!-- プログレスバー -->
        <v-card class="loading-card" :color="'line-white'" elevation="4" :style="{ top: loadingCardTop }">
          <v-card-title>
            <v-icon color="line-green">mdi-robot</v-icon>
            <span class="black--text"> ChatGPT</span>
          </v-card-title>
          <v-card-text class="primary--text" align="center">
            <v-progress-linear indeterminate></v-progress-linear>
            <span>&nbsp;now Loading…</span>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  props: {
    isSendingMessage: Boolean,
  },
  data() {
    return {
      loadingCardTop: '0px',
    };
  },
  watch: {
    isSendingMessage: {
      immediate: true,
      handler(newValue, oldValue) {
        if (newValue) {
          // isSendingMessageがtrueの場合にのみ、メッセージ要素を取得
          const collection = document.getElementsByClassName('message');
          const maxIndexElement = collection[collection.length - 1];

          // 要素の位置情報を取得
          const rect = maxIndexElement.getBoundingClientRect();
          console.log('要素の高さ:', rect.height);

          // 表示位置を動的に指定
          this.loadingCardTop = `-${rect.height + 24}px`;
        }
      }
    }
  },
};
</script>

<style scoped>
.loading-card {
  height: 96.3%;
}
</style>
