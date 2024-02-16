<template>
  <v-container>
    ChatForm<v-icon>mdi-apps</v-icon>
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
      newMyMessage: "",
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
      if (this.newMyMessage.trim() === "") {
        return;
      }

      try {
        // OpenAI APIにメッセージを送信してレスポンスを受け取る
        const apiEndpoint = 'https://api.openai.com/v1/chat/completions';
        const apiKey = process.env.OPENAI_API_KEY;
        const gptMessages = this.messages.filter(message => message.who === "Gpt");
        const lastGptMessage = gptMessages.length > 0 ? gptMessages[gptMessages.length - 1].msg : '';
        const response = await fetch(apiEndpoint, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${apiKey}`
          },
          body: JSON.stringify({
            "model": "gpt-3.5-turbo",
            "messages": [
              { "role": "assistant", "content": lastGptMessage }, // 文脈に沿った回答をさせるために必要
              { "role": "user", "content": this.newMyMessage }
            ],
            "temperature": 1.0, // 回答のランダム度合い：値が高いほど多様な単語が選ばれる（0〜1で指定）
            "max_tokens": 300,  // 回答文章のトークンの最大数（最大4096）
            "stream": true,     // ストリーミング
          })
        });

        // エラー処理
        if (response.status !== 200) {
          throw new Error(`HTTPエラー！ステータス：${response.status}`);
        }

        // ユーザーのメッセージをチャットに追加しておく
        this.messages.push({ who: "Me", msg: this.newMyMessage });

        // ストリーミングで応答を処理
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let responseData = '';
        while (true) {
          const { done, value } = await reader.read();

          if (done) {
            // 一番下までスクロール
            window.scrollTo(0, document.body.scrollHeight);

            console.log('ストリーミングレスポンスが完了しました。');
            break;
          }

          responseData += decoder.decode(value);

          // 改行で応答データを分割し、それぞれの部分に対して処理を行う
          const responses = responseData.split('\n');
          for await (const partialResponse of responses.slice(0, -1)) {
            const match = partialResponse.match(/"content":"([^"]*)"/);
            const content = match ? match[1] : null;

            if (content) {
              // ChatGPTからのメッセージを格納する
              const getWho = this.messages[this.messages.length - 1].who;
              if (getWho !== "Gpt") {
                this.messages.push({ who: "Gpt", msg: content });
              } else {
                this.messages[this.messages.length - 1].msg += content;
              }
              this.newMyMessage = '';
            }
          }

          // 未完了の部分を残して次の読み込みに備える
          responseData = responses[responses.length - 1];

          // 一番下までスクロール
          window.scrollTo(0, document.body.scrollHeight);
        }
      } catch (error) {
        console.error("Error sending message:", error);
      }
    },
  },
};
</script>

<style scoped></style>
