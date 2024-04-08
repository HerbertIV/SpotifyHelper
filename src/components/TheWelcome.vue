<script setup>

import {ref, watch} from "vue";
import axios from "axios";
import Papa from 'papaparse';

const file = ref();
const data = ref([]);
const upload = async (e) => {
  let singleFile = file.value.files[0];
  await Papa.parse(singleFile, {
    complete: async function (results) {
      let headers = [];
      let max = process.env.MAX_LIMIT <= results.data.length ? process.env.MAX_LIMIT : results.data.length;
      for (let i = 0; i < max; i++) {
        let element = results.data[i];
        let row = [];
        for (let j = 0; j < element.length; j++) {
          if (i === 0) headers.push(element[j]); else row[headers[j]] = element[j];
        }
        if (row && i > 0) {
          if (typeof row['Link'] !== undefined && row['Link']) {
            const result = await fetchRapid(row['Link'].replace(/.*\/(\w+)$/g, '$1'));
            let monthlyListeners = row['Monthly Listeners'] === '' ? 0 : parseInt(row['Monthly Listeners']);
            row['Zmiana ML'] = result.data.monthly_listeners - monthlyListeners;
            data.value.push({
              ...row,
              ...{'Monthly Listeners': result.data.monthly_listeners}
            });
          }
        }
      }
      data.value.sort(
        (a, b) => a['Monthly Listeners'] > b['Monthly Listeners'] ?
          -1 :
          (a['Monthly Listeners'] < b['Monthly Listeners'] ? 1 : 0)
      );
      updatePosition(data);
    }
  });
}

const updatePosition = (data) => {
  let updateData = [];
  for (let i = 0; i < data.value.length; i++) {
    let row = data.value[i];
    let position = row['Pozycja'];
    let newPosition = i+1;
    row['Zmiana pozycji'] = position - newPosition;
    updateData.push({
      ...row,
      ...{'Pozycja': newPosition}
    })
  }

  data.value = updateData;
}

const downloadFile = () => {
  let now = new Date(Date.now());
  let fileDate = now.getDate() + '' + now.getMonth() + '' + now.getFullYear();
  const csv = new Blob([Papa.unparse(data.value)], { type: 'text/csv' });
  const url = window.URL.createObjectURL(csv);
  const link = document.createElement('a');
  link.href = url;
  link.setAttribute('download', 'TOP_CURRENTLY_' + fileDate);
  document.body.appendChild(link);
  link.click();
}

const fetchRapid = async (artistId) => {
  if (process.env.MODE === 'production') {
    const options = {
      method: 'GET',
      url: process.env.RAPID_API_URL,
      params: {
        spotify_artist_id: artistId
      },
      headers: {
        'X-RapidAPI-Key': process.env.RAPID_API_KEY,
        'X-RapidAPI-Host': process.env.RAPID_API_HOST
      }
    };

    try {
      const response = await axios.request(options);

      return {
        data: response.data
      };
    } catch (error) {
      console.error(error);
    }
  }

  return {
    data: {
      message: "Data Retrieved.",
      monthly_listeners: Math.floor(Math.random() * 30000),
      result: "success",
      spotify_artist_id: "4lS1f07YGBEKA9ai638UbU"
    }
  };
}
</script>

<template>
  <div class="space-y-12">
    <div class="border-b border-gray-900/10 pb-12">
      <div class="mt-10 grid grid-cols-1 gap-x-6 gap-y-8 sm:grid-cols-6">
        <div class="col-span-full">
          <label for="photo" class="block text-sm font-medium leading-6 text-gray-900">Upload</label>
          <div class="mt-2 flex items-center gap-x-3">
              <input type="file" @change="upload" ref="file">
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="mt-6 flex items-center justify-end gap-x-6">
    <a
        class="rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
        v-if="data.length > 0"
        @click.prevent="downloadFile()" >
      Pobierz
    </a>
  </div>
</template>
