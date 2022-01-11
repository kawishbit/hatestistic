<template>
	<transition name="modal-fade">
		<div class="modal-backdrop">
			<div
				class="modal"
				role="dialog"
				aria-labelledby="modalTitle"
				aria-describedby="modalDescription"
			>
				<header class="modal-header" id="modalTitle" v-if="header">
					<slot name="header"></slot>
					<span
						class="modal-close"
						@click="close"
						aria-label="Close modal"
						><i class="fas fa-times"></i
					></span>
				</header>

				<section class="modal-body" id="modalDescription">
					<slot name="body"> </slot>
				</section>

				<footer class="modal-footer" v-if="footer">
					<slot name="footer"></slot>
				</footer>
			</div>
		</div>
	</transition>
</template>
<script>
export default {
	name: "Modal",
	props: {
		header: {
			type: Boolean,
			default: true,
		},
		footer: {
			type: Boolean,
			default: true,
		},
	},
	methods: {
		close() {
			this.$emit("close");
		},
	},
};
</script>