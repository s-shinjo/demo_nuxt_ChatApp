<template>
  <v-container>
    AssistantForm!<v-icon color="line-green">mdi-robot</v-icon>
    <v-container v-show="!isShowChat">
      <!-- スタートコンポーネント -->
      <StartScreen :start-chat="startChat" />
    </v-container>

    <v-container v-show="isShowChat">
      <!-- チャットメッセージ -->
      <ChatMessage :messages="messages" />

      <!-- 余白 -->
      <v-row>
        <v-col class="mb-10">
        </v-col>
      </v-row>

      <!-- 入力コンポーネント -->
      <ChatInputs :value="newMyMessage" :send-message="sendMessage" @input="updateNewMyMessage" />
    </v-container>
  </v-container>
</template>

<script>
import StartScreen from "@/components/templates/StartScreen.vue";
import ChatMessage from "@/components/templates/ChatMessage.vue";
import ChatInputs from "@/components/templates/ChatInputs.vue";

// APIエンドポイントから非同期でJSONデータを取得する
async function fetchJsonFromAPI(apiKey, apiEndpoint, methodType, requestBody='') {
  const options = {
    method: methodType,
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
      'OpenAI-Beta': 'assistants=v1',
    },
  };
  if (requestBody) {
    options.body = requestBody;
  }

  const response = await fetch(apiEndpoint, options);
  return response.json();
}

export default {
  components: {
    StartScreen,
    ChatMessage,
    ChatInputs,
  },

  data() {
    return {
      isShowChat: false,
      messages: [],
      newMyMessage: '',
    };
  },

  methods: {
    startChat() {
      this.isShowChat = true;
    },
    updateNewMyMessage(value) {
      this.newMyMessage = value;
    },
    async sendMessage(event) {
      // シフト+エンターで改行
      if (event.shiftKey) {
        this.newMyMessage += '\n';
        return;
      }
      // 空の場合は送信しない
      if (this.newMyMessage.trim() === '') {
        return;
      }

      try {
        const assistantId = process.env.OPENAI_ASSISTANT_ID;
        const apiKey = process.env.OPENAI_API_KEY;
        const apiEndpoint = 'https://api.openai.com/v1/threads';
        let requestBody;

        // // 0.これでアシスタント作成できる
        // requestBody = JSON.stringify({
        //   instructions: "平和な人です",
        //   name: "テスト",
        //   tools: [{ type: "code_interpreter" }],
        //   "model": "gpt-3.5-turbo",
        // });
        // const assistant = await fetchJsonFromAPI(apiKey, 'https://api.openai.com/v1/assistants', 'POST', requestBody);
        // console.log('assistant', assistant);

        // 1.スレッドを作成
        const emptyThread = await fetchJsonFromAPI(apiKey, apiEndpoint, 'POST');
        const threadId = emptyThread.id;
        console.log('thread.id:', threadId);

        // // 2.スレッドにメッセージを作成
        requestBody = JSON.stringify({ role: 'user', content: this.newMyMessage });
        const threadMessages = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/messages`, 'POST', requestBody);
        console.log('threadMessages.id:', threadMessages.id);

        this.messages.push({ who: "Me", msg: this.newMyMessage });
        this.newMyMessage = '';

        // 3.（作成済みの）アシスタントを使用しスレッドを実行
        requestBody = JSON.stringify({ 'assistant_id': assistantId });
        const run = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/runs`, 'POST', requestBody);
        console.log('run.id:', run.id);

        // 4.ステータスがcompletedになるまで待機
        let isCompleted = false;
        while (!isCompleted) {
          const runRetrieve = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/runs/${run.id}`, 'GET');
          isCompleted = (runRetrieve.status === 'completed');
          if (!isCompleted) {
            // 1秒間待機
            await new Promise(resolve => setTimeout(resolve, 1000)); // 1000ミリ秒 = 1秒
            console.log('wait');
          }
        }

        // 5.メッセージを取得
        const getMessages = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/messages`, 'GET');
        const newGptMessage = getMessages.data[0].content[0].text.value;
        console.log('getMessagesInfo:', getMessages);
        console.log('newGptMessage:', newGptMessage);
        

        this.messages.push({ who: "Gpt", msg: newGptMessage });

      } catch (error) {
        console.error("Error sending message:", error);
      }
    },
  },
};
</script>

<style scoped></style>
