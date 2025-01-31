<template>
    <div class="slider-container">
        <div 
			class="slider-track" 
			@click="setValue($event)"
		>

            <div 
                class="slider-random" 
                ref="random"
                :style="{left: randomProgress}"
                :class="{'c-green': randomChangeColor}"
                v-if="random > 0"
            >
                <span>{{ random }}</span>
            </div>

			<div class="slider-progress" :style="{ width: progress + '%' }"></div>
			
			<div class="slider-thumb" 
				:style="{ left: thumb }" 
				ref="thumb" 
                @mousedown="startDrag($event)"
				@touchstart="startDrag($event)">
			</div>
		</div>

		<!-- <div>Min: {{ min }}</div>
		<div>Chance: {{ value }}</div>
		<div>Max: {{ max }}</div>
        <div>Rand: {{ random }}</div> -->

		<input type="range" v-model="internalValue" :min="min" :max="max">
    </div>
</template>
  
<script>
export default {
    props: {
        value: {
            type: Number,
            default: 0,
        },
        min: {
            type: Number,
            default: 0,
        },
        max: {
            type: Number,
            default: 100,
        },
        random: {
            type: Number,
            default: 0
        }
    },
    data() {
        return {
            internalValue: this.value, 
            thumb: '0%',
            randomProgress: '0%',
            randomChangeColor: false,
            isDragging: false,
        };
    },
    watch: {
        value(newValue) {
            this.setValueFromProp(newValue);
        },
        internalValue(newValue) {
            this.emitValue(newValue); 
        },
        random(newValue) {
            this.setPositionRandom(newValue);
        }
    },
    methods: {
        setPositionRandom(number) {
            const percentage = ((number - this.min) / (this.max - this.min)) * 100;
            this.randomProgress = `calc(${percentage}% - ${this.$refs.random.offsetWidth / 2}px)`;

            if (percentage > this.value) {
                this.randomChangeColor = true;
            } else {
                this.randomChangeColor = false;
            }
        },

        setValue(event, thumb) {
            const range = (thumb != undefined) ? thumb : event.currentTarget;
            const rect = range.getBoundingClientRect();
            let offsetX = event.clientX - rect.left;

            if (thumb != undefined) {
                offsetX = (event.type.includes('touch') ? event.touches[0].clientX : event.clientX) - rect.left;
            } else {
                offsetX = event.clientX - rect.left;
            }

            const percentage = Math.round((offsetX / rect.width) * 100);

            if (percentage >= 0 && percentage <= 100) {
                this.internalValue = percentage;
                this.progress = percentage;
                this.thumb = `calc(${percentage}% - ${this.$refs.thumb.offsetWidth / 2}px)`;

                if (percentage <= 0) {
                    this.thumb = `calc(${this.min}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
                } else if (percentage >= this.max) {
                    this.thumb = `calc(${this.max}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
                }
            }
        },

        setValueFromProp(percent) {
            this.progress = percent;
            this.thumb = `calc(${percent}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
        },

        emitValue(newValue) {
            this.$emit('update:value', newValue);
        },

        onDrag(event) {
            if (this.isDragging) {
                this.setValue(event, this.$refs.thumb.parentElement);
            }
        },

        startDrag(event) {
            this.isDragging = true;
            document.addEventListener('mousemove', this.onDrag);
            document.addEventListener('mouseup', this.stopDrag);
            document.addEventListener('touchmove', this.onDrag);
            document.addEventListener('touchend', this.stopDrag);
        },

        stopDrag() {
            this.isDragging = false;
            document.removeEventListener('mousemove', this.onDrag);
            document.removeEventListener('mouseup', this.stopDrag);
            document.removeEventListener('touchmove', this.onDrag);
            document.removeEventListener('touchend', this.stopDrag);
        }
    },

    mounted() {
        this.setValueFromProp(this.value); 
    },
};
</script>
  
<style scoped>
.slider-container {
    padding-top: 50px;

	.slider-track {
		height: 8px;
		width: 100%;
		border-radius: 15px;
		position: relative;
		cursor: pointer;
		background: #16d95e;
		margin-top: 30px;
	}

	input[type="range"] {display: none;}

    .slider-random {
        width: 56px;
        height: 56px;
        background-image: url(data:image/svg+xml,%3csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%2045.55%2051.32'%3e%3cpath%20d='M43.28%2011.13L25.05.61a4.56%204.56%200%2000-4.55%200L2.28%2011.13A4.55%204.55%200%20000%2015.07v21a4.55%204.55%200%20002.28%203.94L20.5%2050.58a4.56%204.56%200%20004.55%200l18.23-10.52a4.56%204.56%200%20002.27-3.94v-21a4.56%204.56%200%2000-2.27-3.99z'%20fill='%23fff'/%3e%3cpath%20d='M21.13%2024.64L1.85%2013.51A1.23%201.23%200%20000%2014.57v22a3.69%203.69%200%20001.84%203.19l19.09%2011a3.69%203.69%200%20001.85.49%2034.48%2034.48%200%20000-23.82%203.3%203.3%200%2000-1.65-2.79z'%20fill='%23e9f0f5'/%3e%3cpath%20d='M43.7%2013.51L24.42%2024.64a3.3%203.3%200%2000-1.64%202.86v23.82a3.68%203.68%200%20001.84-.49l19.09-11a3.69%203.69%200%20001.84-3.19v-22a1.23%201.23%200%2000-1.85-1.13z'%20fill='%23d3dee6'/%3e%3c/svg%3e);
        background-repeat: no-repeat;
        position: absolute;
        left: 0;
        bottom: 100%;
        margin-bottom: 10px;
        display: inline-flex;
        align-items: center;
        justify-content: center;
        transition: all 0.3s;
        font-size: 11px;
        font-weight: 500;
        color: red;

        span {
            position: relative;
            top: 2px;
            left: -3px;
        }

        &.c-green {color: #16d95e;}
    }

	.slider-progress {
		background: #ef4444;
		border-radius: 15px;
		height: 100%;
		transition: none !important;
	}

	.slider-thumb {
		top: -6px;
		width: 20px;
		height: 20px;
		background: #0c0b0b;
        border: 2px solid #16d95e;
		border-radius: 50%;
		cursor: pointer;
		position: absolute;
		transition: none !important;
	}
}
</style>
