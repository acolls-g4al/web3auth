<template>
	<Modal />
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Modal from './components/Modal.vue';

export default defineComponent({
	name: 'App',
	components: {
		Modal,
	},
});
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #c533b2;
	margin-top: 60px;
	background: black;
}
html {
	background: black;
	color: #c533b2;
}
</style>
