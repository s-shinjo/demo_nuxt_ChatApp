<template>
  <v-container>
    AssistantForm<v-icon color="line-green">mdi-robot</v-icon>
    <v-container v-show="!isShowChat">
      <!-- スタートコンポーネント -->
      <StartScreen :start-chat="startChat" />
    </v-container>

    <v-container v-show="isShowChat">
      <!-- チャットメッセージ -->
      <ChatMessage :messages="messages" />

      <!-- プログレスバー -->
      <ProgressMessage :is-sending-message="isSendingMessage" :waitingTime="waitingTime" />

      <!-- 余白 -->
      <v-row>
        <v-col class="mb-15">
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
import ProgressMessage from "@/components/templates/ProgressMessage.vue";
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
    ProgressMessage,
    ChatInputs,
  },

  data() {
    return {
      isShowChat: false,
      isSendingMessage: false, // true時、プログレスバーを表示
      waitingTime: 5, // 単位：秒
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

        // 問い合わせ内容を表示
        const myMessage = this.newMyMessage;
        this.messages.push({ who: "Me", msg: myMessage });
        this.newMyMessage = ''; // 連投防止のためここで空にしておく
        this.isSendingMessage = true; // プログレスバー：ON

        // スレッドを作成
        const emptyThread = await fetchJsonFromAPI(apiKey, apiEndpoint, 'POST');
        const threadId = emptyThread.id;
        console.log('thread.id:', threadId);

        // スレッドにメッセージを作成
        requestBody = JSON.stringify({ role: 'user', content: myMessage });
        const threadMessages = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/messages`, 'POST', requestBody);
        console.log('threadMessages.id:', threadMessages.id);

        // （Playgroundで作成済みの）アシスタントを使用しスレッドを実行
        requestBody = JSON.stringify({ 'assistant_id': assistantId });
        const run = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/runs`, 'POST', requestBody);
        console.log('assistantId:', assistantId);
        console.log('run.id:', run.id);

        // ステータスがcompleted or failedになるまで待機
        let isLoop = true;
        while (isLoop) {
          const runRetrieve = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/runs/${run.id}`, 'GET');
          console.log('status:' + runRetrieve.status);
          switch (runRetrieve.status) {
            case 'completed':
              isLoop = false;
              break;
            case 'failed':
              console.log('status last_error:', JSON.stringify(runRetrieve.last_error));
              isLoop = false;
              break;
            default:
              // N秒間待機
              await new Promise(resolve => setTimeout(resolve, this.waitingTime * 1000)); // 1000ミリ秒 = 1秒
              break;
          }
        }

        // メッセージを取得
        const getMessages = await fetchJsonFromAPI(apiKey, apiEndpoint + `/${threadId}/messages`, 'GET');
        let newGptMessage = getMessages.data[0].content[0].text.value;
        
        // 末尾に【N†source】があれば削除
        const index = newGptMessage.indexOf('【');
        if (index !== -1) {
          newGptMessage = newGptMessage.substring(0, index);
        }
        console.log(newGptMessage);

        // レスポンス内容を表示
        this.messages.push({ who: "Gpt", msg: newGptMessage });
        this.isSendingMessage = false; // プログレスバー：OFF

        // 一番下までスクロール
        window.scrollTo(0, document.body.scrollHeight);
      } catch (error) {
        console.error("Error sending message:", error);
      } finally {
        // 念のためfinally
        this.isSendingMessage = false;
      }
    },
  },
};
</script>

<style scoped></style>
