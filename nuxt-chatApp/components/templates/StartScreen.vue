<template>
	<v-container>
		<transition name="fade" appear>
			<!-- スタート画面 -->
			<v-row class="start-screen">
				<v-col cols="8">
					<v-card color="#6c9bd2" elevation="4" style="background-image: url('/vuetify-logo.svg'); background-position: right;">
						<v-card-title class="text-h5">チャットを開始</v-card-title>
						<v-card-text class="text-h6">
							チャットアプリへようこそ！<br>
							ボタンをクリックしてチャットを開始しよう
						</v-card-text>
						<v-card-actions class="justify-center">
							<v-btn color="button-blue" @click="startChat">
								チャット開始<v-icon>mdi-chat-plus</v-icon>
							</v-btn>
						</v-card-actions>
					</v-card>
				</v-col>
			</v-row>
		</transition>
	</v-container>
</template>

<script>
export default {
	props: {
		startChat: Function,
	},
};
</script>

<style scoped>
/***********************/
/***** スタート画面 *****/
/***********************/
.start-screen {
	position: flex;
	margin: 25vh 0 0 0;
	justify-content: center;
	text-align: center;
}

/***********************/
/***** フェードイン *****/
/***********************/
.fade-enter-active,
.fade-leave-active {
	transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
	opacity: 0;
}
</style>
