<script setup>
import {
  Card,
  CardHeader,
  CardTitle,
  CardContent,
  CardFooter,
} from '@/components/ui/card';

import { Input } from '@/components/ui/input';

import { reactive, ref, computed, watch } from 'vue';

const props = defineProps({ scheduleConfig: { type: String, default: null } });

const frequencyOptions = [
  { label: 'Weekly', value: 'weekly' },
  { label: 'Fortnightly', value: 'fortnightly' },
  { label: 'Every 4 weeks', value: 'every4weeks' },
  { label: 'None', value: 'none' },
  { label: 'Custom', value: 'custom' }
];

const days = [
  { label: 'Mon', value: 'monday' },
  { label: 'Tue', value: 'tuesday' },
  { label: 'Wed', value: 'wednesday' },
  { label: 'Thu', value: 'thursday' },
  { label: 'Fri', value: 'friday' },
  { label: 'Sat', value: 'saturday' },
  { label: 'Sun', value: 'sunday' }
];

const frequency = ref('fortnightly');
const selectedDays = ref([]);
const useTimeWindow = ref(false);
const timeWindow = reactive({ start: '', end: '' });
const timeWindowError = ref('');
const useExactTime = ref(false);
const exactTime = ref('');

if (props.scheduleConfig) {
  try {
    const parsed = JSON.parse(props.scheduleConfig);
    frequency.value = parsed.frequency || 'fortnightly';
    selectedDays.value = parsed.days || [];
    if (parsed.timeWindow) {
      useTimeWindow.value = true;
      timeWindow.start = parsed.timeWindow.start;
      timeWindow.end = parsed.timeWindow.end;
    }
    if (parsed.exactTime) {
      useExactTime.value = true;
      exactTime.value = parsed.exactTime;
    }
  } catch { console.warn('Invalid scheduleConfig JSON'); }
}

function validateTimeWindow() {
  if (!useTimeWindow.value) { timeWindowError.value = ''; return; }
  if (!timeWindow.start) { timeWindowError.value = 'Start time is required'; return; }
  if (!timeWindow.end) { timeWindowError.value = 'End time is required'; return; }
  if (timeWindow.start >= timeWindow.end) { timeWindowError.value = 'Start time must be before end time'; return; }
  timeWindowError.value = '';
}

function clearTimeWindow() {
  if (useExactTime.value) {
    useTimeWindow.value = false;
    timeWindow.start = '';
    timeWindow.end = '';
    timeWindowError.value = '';
  }
}

function clearExactTime() {
  if (useTimeWindow.value) {
    useExactTime.value = false;
    exactTime.value = '';
  }
}

const scheduleConfigJson = computed(() => {
  const config = { frequency: frequency.value, days: selectedDays.value };
  if (useTimeWindow.value && !timeWindowError.value) config.timeWindow = { ...timeWindow };
  if (useExactTime.value) config.exactTime = exactTime.value;
  return JSON.stringify(config, null, 2);
});

const jsonInput = ref('');

function applyJsonInput() {
  if (!jsonInput.value.trim()) return;
  
  try {
    const parsed = JSON.parse(jsonInput.value);
    
    // Apply frequency
    if (parsed.frequency && frequencyOptions.some(opt => opt.value === parsed.frequency)) {
      frequency.value = parsed.frequency;
    }
    
    // Apply days
    if (parsed.days && Array.isArray(parsed.days)) {
      selectedDays.value = parsed.days.filter(day => 
        days.some(d => d.value === day)
      );
    }
    
    // Apply time window
    if (parsed.timeWindow && parsed.timeWindow.start && parsed.timeWindow.end) {
      useTimeWindow.value = true;
      timeWindow.start = parsed.timeWindow.start;
      timeWindow.end = parsed.timeWindow.end;
      useExactTime.value = false;
      exactTime.value = '';
    }
    
    // Apply exact time
    if (parsed.exactTime && !parsed.timeWindow) {
      useExactTime.value = true;
      exactTime.value = parsed.exactTime;
      useTimeWindow.value = false;
      timeWindow.start = '';
      timeWindow.end = '';
    }
    
    // Clear the input after successful application
    jsonInput.value = '';
    
  } catch (error) {
    console.error('Invalid JSON input:', error);
    alert('Invalid JSON format. Please check your input.');
  }
}

function clearAll() {
  frequency.value = 'fortnightly';
  selectedDays.value = [];
  useTimeWindow.value = false;
  timeWindow.start = '';
  timeWindow.end = '';
  timeWindowError.value = '';
  useExactTime.value = false;
  exactTime.value = '';
  jsonInput.value = '';
}

function showSampleJson() {
  const sample = {
    frequency: 'custom',
    days: ['monday', 'wednesday', 'friday'],
    timeWindow: {
      start: '09:00',
      end: '17:00'
    }
  };
  jsonInput.value = JSON.stringify(sample, null, 2);
}

watch([() => timeWindow.start, () => timeWindow.end, useTimeWindow], validateTimeWindow, { immediate: true });
watch(selectedDays, (newVal) => {
  if (newVal.length === 0) {
    useTimeWindow.value = false;
    useExactTime.value = false;
    timeWindow.start = '';
    timeWindow.end = '';
    exactTime.value = '';
    timeWindowError.value = '';
  }
});

// Watch all form fields to ensure JSON stays in sync
watch([frequency, selectedDays, useTimeWindow, useExactTime, () => timeWindow.start, () => timeWindow.end, exactTime], () => {
  // The computed property will automatically update when any of these change
}, { deep: true });

// Watch frequency changes to clear custom-specific fields when switching away from custom
watch(frequency, (newFrequency) => {
  if (newFrequency !== 'custom') {
    // Clear custom-specific fields when not using custom frequency
    selectedDays.value = [];
    useTimeWindow.value = false;
    timeWindow.start = '';
    timeWindow.end = '';
    timeWindowError.value = '';
    useExactTime.value = false;
    exactTime.value = '';
  }
});
</script>

<template>
  <Card>
    <CardHeader>
      <CardTitle>Service Frequency</CardTitle>
    </CardHeader>
    <CardContent>
      <div class="grid grid-cols-2 gap-6">
        <!-- Left Column: Inputs -->
        <div class="space-y-4">
          <div class="space-y-2">
            <h4 class="font-medium">Frequency Options</h4>
            <div v-for="opt in frequencyOptions" :key="opt.value" class="flex items-center space-x-2">
              <input type="radio" v-model="frequency" :value="opt.value" :id="`freq-${opt.value}`" />
              <label :for="`freq-${opt.value}`" class="text-sm">{{ opt.label }}</label>
            </div>
          </div>

          <div v-if="frequency === 'custom'" class="space-y-4">
            <div>
              <h4 class="font-medium mb-2">Select Days</h4>
              <div class="grid grid-cols-2 gap-2">
                <label v-for="day in days" :key="day.value" class="flex items-center space-x-2">
                  <input
                    type="checkbox"
                    v-model="selectedDays"
                    :value="day.value"
                    :id="day.value"
                  />
                  <span class="text-sm">{{ day.label }}</span>
                </label>
              </div>
            </div>

            <div class="space-y-3">
              <label class="flex items-center space-x-2">
                <input
                  type="checkbox"
                  v-model="useTimeWindow"
                  :disabled="selectedDays.length === 0"
                  @change="clearExactTime"
                />
                <span class="text-sm">Specify time window</span>
              </label>
              <div v-if="useTimeWindow" class="ml-6 space-y-2">
                <div>
                  <label class="block text-sm font-medium mb-1">Start Time</label>
                  <Input type="time" v-model="timeWindow.start" />
                </div>
                <div>
                  <label class="block text-sm font-medium mb-1">End Time</label>
                  <Input type="time" v-model="timeWindow.end" />
                </div>
                <div v-if="timeWindowError" class="text-red-500 text-sm">{{ timeWindowError }}</div>
              </div>

              <label class="flex items-center space-x-2">
                <input
                  type="checkbox"
                  v-model="useExactTime"
                  :disabled="selectedDays.length === 0"
                  @change="clearTimeWindow"
                />
                <span class="text-sm">Set exact time</span>
              </label>
              <div v-if="useExactTime" class="ml-6">
                <label class="block text-sm font-medium mb-1">Exact Time</label>
                <Input type="time" v-model="exactTime" />
              </div>
            </div>
          </div>
        </div>

        <!-- Right Column: JSON Display -->
        <div class="space-y-4">
          <h4 class="font-medium">Schedule Configuration</h4>
          <div class="bg-gray-50 p-4 rounded-lg border">
            <pre class="text-sm text-gray-800 whitespace-pre-wrap">{{ scheduleConfigJson }}</pre>
          </div>
          <div class="mt-4">
            <h4 class="font-medium mb-2">Paste JSON Configuration</h4>
            <textarea
              v-model="jsonInput"
              class="w-full p-2 border rounded-md"
              rows="5"
              placeholder="Paste JSON configuration here"
            ></textarea>
            <button
              @click="applyJsonInput"
              class="mt-2 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600"
            >
              Apply JSON
            </button>
            <button
              @click="clearAll"
              class="mt-2 ml-2 px-4 py-2 bg-red-500 text-white rounded-md hover:bg-red-600"
            >
              Clear All
            </button>
            <button
              @click="showSampleJson"
              class="mt-2 ml-2 px-4 py-2 bg-purple-500 text-white rounded-md hover:bg-purple-600"
            >
              Show Sample JSON
            </button>
          </div>
        </div>
      </div>
    </CardContent>
  </Card>
</template>
