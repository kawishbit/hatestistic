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
		<textarea v-model="mainText" placeholder="Your text here..."></textarea>
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
				@click="resetSystem()"
			>
				<i class="fas fa-sync"></i>
			</button>
			<button
				class="btn-icon btn-info"
				:disabled="!calculated || disableButtons"
			>
				<i class="fas fa-exclamation"></i>
			</button>
			<button
				class="btn-icon btn-primary"
				:disabled="calculated || disableButtons"
				@click="getResult()"
			>
				<i class="fas fa-arrow-right"></i>
			</button>
		</div>
	</div>
</template>

<script>
import { defineComponent } from "vue";
import axios from "axios";
import Number from "@/components/Number.vue";

export default defineComponent({
	name: "Home",
	components: {
		Number,
	},
	data() {
		return {
			disableButtons: false,
			numberFrom: 0,
			duration: 3,
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
				is_hate_speech: false,
			},
			randomTexts: [
				"Jokowi tabang",
				"Jokowi salto",
				"Jokowi jokowi",
				"What jokowi",
				"Why jokowi",
				"Whomst Jokowi",
				"Where jokowi",
				"Wagwan G",
				"Ya dunkno fam",
				"Sheeesh",
				"Lorem Ipsum",
				"You takin the piss?",
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
				let resp = await axios.get(
					"https://api.sampleapis.com/coffee/hot"
				);
				console.log(resp);
			} catch (err) {
				throw new Error("Error");
			}

			this.numberFrom = 0;
			this.result = {
				q: "asdasd",
				filtered: "asdasdasd",
				prediction: 98.22,
				is_hate_speech: true,
			};

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
				is_hate_speech: false,
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
			}, 4000);
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
	},
});
</script>
