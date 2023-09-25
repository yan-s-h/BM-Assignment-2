<template>
  <v-container>
    <v-row v-for="(item, index) in data" :key="index">
      <v-col cols="2" class="title">{{ data[index].days }}</v-col>
      <v-col cols="3"><v-switch v-model="data[index].models" hide-details inset
          :label="data[index].models ? '本日供餐' : '本日不供餐'" @click="resetProperties"></v-switch></v-col>
      <v-col cols="3" v-if="data[index].models">

        <v-select label="請選擇時間(Start)" :items="data[index].startTime" v-model="data[index].selectedStartTime"
          variant="outlined" @change="handleEndTimeChange"></v-select></v-col>

      <v-col cols="3" v-if="data[index].models">

        <v-select label="請選擇時間(End)" :items="data[index].endTime" v-model="data[index].selectedEndTime" variant="outlined"
          @change="handleStartTimeChange"></v-select>
      </v-col>
    </v-row>
    <v-row>
      <div>資料如下:</div>
      <v-col cols="12" v-for="(item, index) in data" :key="index">{{ `week_day${index}` }}: "{{
        data[index][`week_day${index}`]
      }} "</v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue';
import { reactive, computed, watch } from 'vue'


const generateTimeIntervals = () => {
  const intervals = [];
  for (let hour = 0; hour < 24; hour++) {
    for (let minute = 0; minute < 60; minute += 30) {
      const formattedHour = hour.toString().padStart(2, '0');
      const formattedMinute = minute.toString().padStart(2, '0');
      intervals.push(`${formattedHour}:${formattedMinute}`);
    }
  }
  return intervals;
};

const startTime = generateTimeIntervals();
const endTime = JSON.parse(JSON.stringify(startTime))
endTime.splice(0, 1)
endTime.push('23:59')
const data = reactive([])
const days = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
for (let i = 0; i < 7; i++) {
  data.push({
    days: days[i],
    models: true,
    startTime: JSON.parse(JSON.stringify(startTime)),
    endTime: JSON.parse(JSON.stringify(endTime)),
    selectedStartTime: ref(''),
    selectedEndTime: ref(''),
    [`week_day${i}`]: "000000000000000000000000000000000000000000000000",
  })
}

console.log(data)
const handleEndTimeChange = (newVal, index) => {
  console.log(`handleStartTimeChange triggered with value ${newVal} at index ${index}`)
};

const handleStartTimeChange = (newVal, index) => {
  console.log(`handleStartTimeChange triggered with value ${newVal} at index ${index}`)
};
data.forEach((item, index) => {
  watch(() => item.selectedStartTime, (newVal) => handleEndTimeChange(newVal, index));
  watch(() => item.selectedEndTime, (newVal) => handleStartTimeChange(newVal, index));
});
const resetItemProperties = (item) => {
  item.startTime = JSON.parse(JSON.stringify(startTime));
  item.endTime = JSON.parse(JSON.stringify(endTime));
  item.selectedStartTime = '';
  item.selectedEndTime = '';
  item[`week_day${data.indexOf(item)}`] = "000000000000000000000000000000000000000000000000";
};


const resetProperties = computed(() => {
  return data.filter((item) => !item.models).forEach((item) => resetItemProperties(item));
});
for (let i = 0; i < 7; i++) {
  const originalStartTime = JSON.parse(JSON.stringify(data[i].startTime));
  const originalEndTime = JSON.parse(JSON.stringify(data[i].endTime));

  watch(() => data[i].selectedStartTime, (newValue, oldValue) => {
    if (newValue !== oldValue) {
      data[i].startTime = originalStartTime.slice();
      data[i].endTime = originalEndTime.slice();
      const startIndex = data[i].startTime.indexOf(newValue);
      if (startIndex >= 0) {
        data[i].endTime.splice(0, startIndex)

        const startIndexForData = originalStartTime.indexOf(newValue) > -1 ? originalStartTime.indexOf(newValue) : 0
        const endIndexForData = originalEndTime.indexOf(data[i].selectedEndTime) > -1 ? originalEndTime.indexOf(data[i].selectedEndTime) : 47
        data[i][`week_day${i}`] =
          "0".repeat(startIndexForData) +
          "1".repeat(endIndexForData - startIndexForData + 1) +
          "0".repeat(48 - endIndexForData - 1);
        console.log(data[i][`week_day${i}`])
      }
    }
  });

  watch(() => data[i].selectedEndTime, (newValue, oldValue) => {
    if (newValue !== oldValue) {
      data[i].startTime = originalStartTime.slice();
      data[i].endTime = originalEndTime.slice();
      const endIndex = data[i].endTime.indexOf(newValue);
      if (endIndex >= 0) {
        data[i].startTime.splice(endIndex + 1);

        const startIndexForData = originalStartTime.indexOf(data[i].selectedStartTime) > -1 ? originalStartTime.indexOf(data[i].selectedStartTime) : 0
        const endIndexForData = originalEndTime.indexOf(newValue) > -1 ? originalEndTime.indexOf(newValue) : 47
        data[i].endTime.splice(0, startIndexForData)
        data[i][`week_day${i}`] =
          "0".repeat(startIndexForData) +
          "1".repeat(endIndexForData - startIndexForData + 1) +
          "0".repeat(48 - endIndexForData - 1);
        console.log(data[i][`week_day${i}`])
      }
    }
  });

}
</script>

<style scoped lang="sass">
.v-row
  height: 102px
.title
  line-height: 60px
</style>
