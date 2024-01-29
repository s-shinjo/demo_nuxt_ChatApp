<template>
	<v-container>
		<v-row v-for="(messageGroup, index) in setMessageGroups" :key="index" cols="12">
        <v-col v-for="(message, messageIndex) in messageGroup" :key="messageIndex" cols="6">
          <v-card class="message" :color="message.who === 'Me' ? 'line-green' : 'line-white'" elevation="4">
            <v-card-title>
              <v-icon v-if="message.who === 'Me'" color="line-white">mdi-human-greeting-variant</v-icon>
              <v-icon v-else color="line-green">mdi-robot</v-icon>
              <span v-if="message.who === 'Me'" class="line-white--text"> Me</span>
              <span v-else class="black--text"> ChatGPT</span>
            </v-card-title>
            <v-card-text class="black--text">{{ formatMessage(message.msg) }}</v-card-text>
          </v-card>
        </v-col>
      </v-row>
	</v-container>
</template>

<script>
export default {
	props: {
    messages: Array,
  },

  computed: {
    setMessageGroups() {
      // 2つずつメッセージをグループ化する
      return this.messages.reduce((groups, message, index) => {
        if (index % 2 === 0) {
          groups.push([message]);
        } else {
          groups[groups.length - 1].push(message);
        }
        return groups;
      }, []);
    },
  },

  methods: {
    formatMessage(message) {
      // メッセージ内の "\\n" を通常の改行文字 "\n" に変換
      return message.replace(/\\n/g, '\n');
    },
  },
};
</script>

<style scoped>
/*****************************/
/***** チャットメッセージ *****/
/*****************************/
.message {
  white-space: pre-line; /* 改行の保持 */
}
</style>