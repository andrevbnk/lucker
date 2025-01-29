<template>
    <div class="slider-container">
        <div 
			class="slider-track" 
			@click="setValue($event)"
		>
			<div class="slider-progress" :style="{ width: progress + '%' }"></div>
			
			<div class="slider-thumb" 
				:style="{ left: thumb }" 
				ref="thumb" 
                @mousedown="startDrag($event)"
				@touchstart="startDrag($event)">
			</div>
		</div>

		<div>Min: {{ min }}</div>
		<div>Chance: {{ value }}</div>
		<div>Max: {{ max }}</div>

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
        }
    },
    data() {
        return {
            internalValue: this.value, 
            thumb: '0%',
            isDragging: false,
        };
    },
    watch: {
        value(newValue) {
            this.setValueFromProp(newValue);
        },
        internalValue(newValue) {
            this.emitValue(newValue); 
        }
    },
    methods: {
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
                this.internalValue = this.min + ((this.max - this.min) * percentage) / 100;
                this.progress = percentage;
                this.thumb = `calc(${percentage}% - ${this.$refs.thumb.offsetWidth / 2}px)`;

                if (percentage <= 0) {
                    this.thumb = `calc(${this.min}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
                } else if (percentage >= this.max) {
                    this.thumb = `calc(${this.max}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
                }
            }
        },

        setValueFromProp(newValue) {
            const percentage = ((newValue - this.min) / (this.max - this.min)) * 100;
            this.progress = percentage;
            this.thumb = `calc(${percentage}% - ${this.$refs.thumb.offsetWidth / 2}px)`;
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
	.slider-track {
		height: 20px;
		width: 100%;
		border-radius: 10px;
		position: relative;
		cursor: pointer;
		background: #db514e;
		margin-top: 30px;
	}
	

	input[type="range"] {display: none;}

	.slider-progress {
		background: #7eb23b;
		border-radius: 10px;
		height: 100%;
		transition: none !important;
	}

	.slider-thumb {
		top: -5px;
		width: 30px;
		height: 30px;
		background: #3b74e6;
		border-radius: 50%;
		cursor: pointer;
		position: absolute;
		transition: none !important;
	}
}
</style>
