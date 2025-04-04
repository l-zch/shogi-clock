<template>
    <div class="w-full max-w-[100vh] relative aspect-square">
        <div
            class="box-content border-3 aspect-square h-full rounded-[50%] flex bg-gray-50 inset-shadow relative"
            :class="is_running ? 'border-gray-200' : 'border-gray-100'"
        >
            <div
                class="w-full h-full absolute rounded-full z-0"
                style="
                    mask-image: radial-gradient(circle, black 0%, black 100%);
                "
            >
                <div
                    class="w-full h-full z-0 drop-shadow-lg transition-all"
                    :style="{
                        background: `conic-gradient(transparent 0deg var(--degree), ${
                            alarm && is_running
                                ? 'var(--color-red-500)'
                                : 'white'
                        } var(--degree) 360deg)`,
                        '--degree': `${degree}deg`,
                        transition: `--degree ${UI_UPDATE_INTERVAL}ms linear`,
                    }"
                ></div>
            </div>
            <div
                :class="[
                    'border-2 aspect-square bg-white h-[75%] rounded-full m-auto z-1 transition-all grid grid-rows-3 place-items-center font-thin',
                    timeout
                        ? 'border-red-500 border-15 md:border-20'
                        : alarm
                        ? 'border-red-500 border-5 scale-110'
                        : is_running
                        ? 'border-gray-600'
                        : 'border-gray-50',
                    is_running ? 'drop-shadow-lg' : 'drop-shadow-sm scale-95',
                ]"
            >
                <div
                    :class="[
                        'text-4xl sm:text-7xl md:text-8xl lg:text-9xl transition-all red-200 row-start-2',
                        timeout
                            ? 'text-red-500 font-bold sm:font-normal sm:text-6xl'
                            : alarm
                            ? 'text-red-500 font-normal scale-130'
                            : is_running
                            ? 'text-black'
                            : 'text-gray-700',
                    ]"
                    :key="time_string"
                >
                    {{ timeout ? "時間切" : time_string }}
                </div>

                <div
                    class="text-gray-400 md:text-4xl lg:text-5xl sm:text-3xl text-2xl row-start-3 self-start"
                    v-show="!timeout"
                >
                    <slot></slot>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref } from "vue";
import {
    CountDownTimer,
    IncrementalTimer,
    UI_UPDATE_INTERVAL,
    Event,
} from "@/timer/timer";

/**
 * @type {{
 *     timer: (CountDownTimer | IncrementalTimer);
 *     alarm: boolean;
 * }}
 */
const { timer, alarm } = defineProps({
    timer: { type: Object },
    alarm: { type: Boolean, default: false },
});

const timeout = ref(false);
const degree = ref(0);
const is_running = ref(timer.is_running);

const remaining_time = ref(timer.remaining_time);

timer.broadcast.add_listener(Event.UPDATE, () => {
    remaining_time.value = timer.remaining_time;
    degree.value =
        (1 -
            (timer.remaining_time -
                (timer.is_running ? UI_UPDATE_INTERVAL : 0)) /
                timer.total_time) *
        360;
});

timer.broadcast.add_listener(Event.TIMEOUT, () => {
    timeout.value = true;
});

timer.broadcast.add_listener(Event.STOP, () => {
    is_running.value = false;
});

timer.broadcast.add_listener(Event.START, () => {
    is_running.value = true;
});

const time_string = computed(() => {
    const ms = remaining_time.value;
    const s = Math.floor(ms / 1000);
    if (s < 60) {
        if (alarm) {
            return `${s}.${Math.floor((ms % 1000) / 100)}`;
        }
        return String(s);
    }

    return `${Math.floor(s / 60)}:${String(s % 60).padStart(2, "0")}`;
});
</script>

<style scoped>
.shadow {
    box-shadow: 2px 2px 6px 0 rgba(0, 0, 0, 0.25);
}

.inset-shadow {
    box-shadow: inset 2px 2px 4px 0 rgba(0, 0, 0, 0.25);
}

@property --degree {
    syntax: "<angle>";
    inherits: false;
    initial-value: 0deg;
}
</style>
