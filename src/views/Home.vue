<template>
	<div class="text-layer" :class="{ hide: !calculated }">
		<textarea
			disabled
			v-model="mainText"
			placeholder="Your text here..."
		></textarea>
	</div>
	<div v-if="calculated" class="overlay-layer" :style="overlayStyle">
		<textarea
			disabled
			v-model="mainText"
			placeholder="Your text here..."
		></textarea>
	</div>
	<div class="main-layer" :class="{ hide: calculated }">
		<textarea
			v-model="mainText"
			placeholder="Your text here..."
			:disabled="calculated"
		></textarea>
	</div>

	<div class="action-layer">
		<div class="action-section">
			<span class="percentage text-dark">
				Hate speech:
				<span class="text-danger">
					<number
						ref="percentage"
						:from="numberFrom"
						:format="theFormat"
						:to="result.prediction"
						:duration="duration"
						easing="Power4.easeOut"
						:delay="1"
					/>
					%</span
				>
			</span>
		</div>
		<div class="action-section fixed">
			<button
				class="btn-icon btn-warning"
				@click="generateText"
				:disabled="calculated || disableButtons"
			>
				<i class="fas fa-quote-right"></i>
			</button>
			<button
				class="btn-icon btn-danger"
				:disabled="!calculated || disableButtons"
				@click="resetSystem"
			>
				<i class="fas fa-sync"></i>
			</button>
			<button
				class="btn-icon btn-info"
				:disabled="!calculated || disableButtons"
				@click="showModal"
			>
				<i class="fas fa-exclamation"></i>
			</button>
			<button
				class="btn-icon btn-primary"
				:disabled="calculated || disableButtons"
				@click="getResult"
			>
				<i class="fas fa-arrow-right"></i>
			</button>
		</div>
	</div>
	<Modal v-show="isDetail" :header="true" :footer="false" @close="closeModal">
		<template v-slot:header> Detail </template>

		<template v-slot:body>
			<p><b>Query :</b> {{ result.q }}</p>
			<p><b>Filtered :</b> {{ result.filtered }}</p>
			<p><b>Prediction :</b> {{ result.predictionPrecise }}</p>
			<p><b>IsHateSpeech :</b> {{ result.isHateSpeech }}</p>
		</template>
	</Modal>
</template>

<script>
import { defineComponent } from "vue";
import axios from "axios";
import Number from "@/components/Number.vue";
import Modal from "@/components/Modal.vue";

export default defineComponent({
	name: "Home",
	components: {
		Number,
		Modal,
	},
	data() {
		return {
			disableButtons: false,
			isDetail: false,
			numberFrom: 0,
			duration: 1.4,
			mainText: "",
			calculated: false,
			overlayStyle: {
				backgroundColor: "#4ade80",
				clipPath: "inset(0 100% 0 0)",
			},
			result: {
				q: "",
				filtered: "",
				prediction: 0,
				predictionPrecise: 0,
				isHateSpeech: false,
			},
			randomTexts: [
				"Jokowi mending mati saja sana",
				"Saya suka neapolitan ice cream",
				"Orang kulit hitam tidak pantas diperlakukan seperti manusia",
				"Kinerja pemerintahan jokowi mungkin bisa ditingkatkan lagi",
				"Usir semua imigran dari tanah kita",
				"China bangsat mengganggu kehidupan org aja",
				"Itu ngapain cebong pada ngumpul di monas, mending tiduran aja dirumah",
				"Kaum cebong kapir udah keliatan dongoknya dari awal tambah dongok lagi",
				"Lawan bicara gw gak intelek kyk loe",
				"Produk ASUS mudah rusak",
				"Video documentary LEMMiNO luar biasa",
				"Saat orang orang saling menuding antek aseng",
			],
			colors: ["#4ade80", "#60a5fa", "#ff9028", "#ef4444"],
		};
	},
	methods: {
		theFormat(number) {
			return number.toFixed(2);
		},
		async getResult() {
			this.disableButtons = true;
			try {
				let resp = await axios.get("https://bern.ibnuhx.com/predict", {
					headers: {
						Accept: "application/json",
					},
					params: {
						q: this.mainText,
					},
				});
				console.log(resp);
				this.numberFrom = 0;
				this.result = {
					q: resp.data.q,
					filtered: resp.data.filtered,
					prediction: parseFloat(resp.data.prediction_percentage),
					predictionPrecise: resp.data.prediction,
					isHateSpeech: resp.data.is_hate_speech,
				};
				console.log(this.result);

				this.overlayStyle = {
					backgroundColor: "#4ade80",
					clipPath: "inset(0 100% 0 0)",
				};

				this.calculated = true;
				setTimeout(() => {
					this.overlayStyle = {
						backgroundColor: this.getColor(),
						clipPath: `inset(0 ${
							100 - Math.round(this.result.prediction)
						}% 0 0)`,
					};
					this.$refs.percentage.restart();
					this.disableButtons = false;
				}, 100);
			} catch (err) {
				console.log(err);
			}
			// this.numberFrom = 0;
			// this.result = {
			// 	q: "asdasd",
			// 	filtered: "asdasdasd",
			// 	prediction: 98.22,
			// 	predictionPrecise: 98.22,
			// 	isHateSpeech: true,
			// };

			// this.overlayStyle = {
			// 	backgroundColor: "#4ade80",
			// 	clipPath: "inset(0 100% 0 0)",
			// };

			// this.calculated = true;
			// setTimeout(() => {
			// 	this.overlayStyle = {
			// 		backgroundColor: this.getColor(),
			// 		clipPath: `inset(0 ${
			// 			100 - Math.round(this.result.prediction)
			// 		}% 0 0)`,
			// 	};
			// 	this.$refs.percentage.restart();
			// 	this.disableButtons = false;
			// }, 100);
		},
		generateText() {
			let text =
				this.randomTexts[
					Math.floor(Math.random() * this.randomTexts.length)
				];
			this.mainText = text;
		},
		resetSystem() {
			this.disableButtons = true;
			this.numberFrom = this.result.prediction;
			this.result = {
				q: "",
				filtered: "",
				prediction: 0.0,
				predictionPrecise: 0.0,
				isHateSpeech: false,
			};

			this.overlayStyle = {
				backgroundColor: "#4ade80",
				clipPath: "inset(0 100% 0 0)",
			};
			this.$refs.percentage.restart();
			setTimeout(() => {
				this.mainText = "";
				this.disableButtons = false;
				this.calculated = false;
			}, 2300);
		},
		getColor() {
			if (this.result.prediction >= 50) {
				return this.colors[3];
			} else if (
				this.result.prediction > 30 &&
				this.result.prediction < 50
			) {
				return this.colors[2];
			} else if (
				this.result.prediction > 20 &&
				this.result.prediction < 30
			) {
				return this.colors[1];
			} else {
				return this.colors[0];
			}
		},
		showModal() {
			this.isDetail = true;
		},
		closeModal() {
			this.isDetail = false;
		},
	},
});
</script>
