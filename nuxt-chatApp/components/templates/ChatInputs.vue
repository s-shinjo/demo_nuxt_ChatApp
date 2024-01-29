<template>
	<v-container>
		<v-row class="input-container">
			<v-col cols="9">
				<v-textarea
					:value="value"
					label="Message ChatGPT…"
					:rows="setDisplayRows"
					@input="$emit('input', $event)"
					@keydown.enter.prevent="sendMessage"
				></v-textarea>
			</v-col>
			<v-col class="send-container" cols="3">
				<v-btn class="send-button" color="button-blue" elevation="4" @click="sendMessage">
					送信<v-icon>mdi-chat-plus</v-icon>
				</v-btn>
				<v-btn fab x-small class="scroll-to-bottom-button" elevation="4" @click="scrollToBottom">
					<v-icon>mdi-arrow-down-bold</v-icon>
				</v-btn>
			</v-col>
		</v-row>
	</v-container>
</template>

<script>
export default {
	props: {
		value: String,
		sendMessage: Function,
	},

	computed: {
		setDisplayRows() {
			// dispMaxRowCount行目まではスクロール無しで表示する
			const currentRowCount = this.value.split('\n').length;
			const dispMaxRowCount = 5;
			return Math.min(currentRowCount, dispMaxRowCount);
		},
	},

	methods: {
		scrollToBottom() {
			// 一番下までスクロール
			window.scrollTo(0, document.body.scrollHeight);
		},
	},
};
</script>

<style scoped>
/*************************/
/***** 入力フィールド *****/
/*************************/
.input-container {
	position: fixed;
	width: 40%;
	min-width: 420px;
	bottom: 20px;
}

.send-container {
	display: flex;
	align-items: flex-end;
	margin: 0 0 0 -20px;
}

.send-button {
	color: #fff;
	border-radius: 8px;
	margin: 0 5px 10px 0;
}

.scroll-to-bottom-button {
	margin: 0 0 10px 0;
}
</style>
