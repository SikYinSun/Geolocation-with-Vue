<template>
  <div class="bg-white p-4 rounded-lg mb-4 max-w-full shadow-md lg:w-[100%]">
    <div class="flex flex-wrap justify-between h-full">
      <div
        v-if="curInfo"
        class="flex-1 bg-gradient-to-r from-blue-400 to-blue-700 rounded-lg flex flex-col items-center justify-center p-4 border border-gray-200 m-2"
      >
        <h2 class="text-white text-xl">Current Location</h2>
        <h1 class="text-white text-2xl mt-2 mb-1">{{ curTime }}</h1>
        <h5 class="text-white text-sm">{{ curInfo.date }}</h5>
        <h5 class="text-white text-sm">{{ curInfo.timezone }}</h5>
      </div>

      <div
        v-if="table.length > 0"
        class="flex-1 bg-gradient-to-r from-sky-400 to-sky-700 rounded-lg flex flex-col items-center justify-center p-4 space-y-2 border border-gray-200 m-2"
      >
        <h2 class="text-white text-xl">{{ table[0].placeName }}</h2>
        <h1 class="text-white text-2xl mt-2 mb-1">{{ targetTime }}</h1>
        <h5 class="text-white text-sm">{{ table[0].date }}</h5>
        <h5 class="text-white text-sm">{{ table[0].timezone }}</h5>
      </div>
    </div>
  </div>

  <CurrentLocation @curUpdate="updateCurLoc" />
  <SearchBar @targetUpdate="targetLoc" />

  <button
    @click="deleteSelected"
    class="ml-4 my-2 py-2 px-4 bg-transparent text-red-600 font-semibold border border-red-600 rounded hover:bg-red-600 hover:text-white hover:border-transparent transition ease-in duration-200 transform hover:-translate-y-1 active:translate-y-0"
  >
    Delete
  </button>

  <div class="max-w-2xl mx-auto">
    <div class="relative overflow-x-auto shadow-md sm:rounded-lg">
      <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="p-4">
              <div class="flex items-center">
                <input
                  type="checkbox"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600"
                />
                <label class="sr-only">checkbox</label>
              </div>
            </th>
            <th scope="col" class="px-6 py-3">Place Name</th>
            <th scope="col" class="px-6 py-3">Time</th>
            <th scope="col" class="px-6 py-3">Date</th>
            <th scope="col" class="px-6 py-3">Timezone</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="location in displayedTable"
            :key="location.id"
            class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-600"
          >
            <td class="w-4 p-4">
              <div class="flex items-center">
                <input
                  v-model="selectedLoc"
                  :value="location"
                  type="checkbox"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600"
                />
                <label class="sr-only">checkbox</label>
              </div>
            </td>
            <th
              scope="row"
              class="px-6 py-4 font-medium text-gray-900 dark:text-white whitespace-nowrap"
            >
              {{ location.placeName }}
            </th>
            <td class="px-6 py-4">
              {{ location.time }}
            </td>
            <td class="px-6 py-4">
              {{ location.date }}
            </td>
            <td class="px-6 py-4">
              {{ location.timezone }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="my-2 flex justify-center items-center">
    <paginate
      v-if="table.length > 0"
      v-model="currentPage"
      :page-count="Math.ceil(table.length / itemsPerPage)"
      :page-range="3"
      :margin-pages="2"
      :click-handler="clickCallback"
      :prev-text="'Prev'"
      :next-text="'Next'"
      :container-class="'pagination'"
    ></paginate>
  </div>

  <GoogleMap
    api-key="AIzaSyCDKfdwkW6ZAGv6eU5ig7IST4e6612onOM"
    style="width: 100%; height: 500px"
    :center="{ lat: tarLoc.lat, lng: tarLoc.lng }"
    :zoom="15"
  >
    <Marker :options="{ position: curLoc, label: 'Current' }" />
    <Marker
      v-for="location in displayedTable"
      :key="location.id"
      :options="{
        position: {
          lat: location.lat,
          lng: location.long,
        },
        label: location.placeName,
      }"
    />
  </GoogleMap>
</template>

<script>
import { ref, defineComponent, watch } from "vue";
import { GoogleMap, Marker } from "vue3-google-map";
import Paginate from "vuejs-paginate-next";
import CurrentLocation from "./CurrentLocation.vue";
import SearchBar from "./SearchBar.vue";

export default defineComponent({
  components: {
    paginate: Paginate,
    GoogleMap,
    Marker,
    CurrentLocation,
    SearchBar,
  },
  setup() {
    const curLoc = ref({ lat: 43.65107, lng: -79.347015 });
    const tarLoc = ref({ lat: 43.65107, lng: -79.347015 });
    const curTime = ref("");
    const curInfo = ref("");
    const targetTime = ref("");
    const table = ref([]);
    const selectedLoc = ref([]);
    const currentPage = ref(1);
    const itemsPerPage = 10;
    const displayedTable = ref([]);

    const getCurrentTime = () => {
      curTime.value = new Date().toLocaleString("en-US", {
        timeZone: curInfo.value.timeID,
        timeStyle: "medium",
      });
    };
    setInterval(getCurrentTime, 1000);

    const getTargetTime = () => {
      if (table.value.length > 0) {
        targetTime.value = new Date().toLocaleString("en-US", {
          timeZone: table.value[0].timeID,
          timeStyle: "medium",
        });
      }
    };
    setInterval(getTargetTime, 1000);

    const updateCurLoc = (data) => {
      curInfo.value = data;
      curLoc.value = { lat: data.lat, lng: data.long };
      tarLoc.value = curLoc.value;
      getCurrentTime();
    };

    const targetLoc = (data) => {
      data.id = Date.now();
      table.value.unshift(data);
      tarLoc.value = { lat: data.lat, lng: data.long };
      if (table.value.length <= 10) {
        displayedTable.value = table.value;
      } else {
        displayedTable.value = table.value.slice(0, 10);
      }
    };

    const deleteSelected = () => {
      if (selectedLoc.value.length > 0) {
        const ids = selectedLoc.value.map((location) => location.id);
        table.value = table.value.filter(
          (location) => !ids.includes(location.id)
        );
        selectedLoc.value = [];

        if (table.value.length <= 10) {
          displayedTable.value = table.value;

          if (table.value.length == 0) {
            tarLoc.value = curLoc.value;
          } else {
            tarLoc.value = {
              lat: displayedTable.value[0].lat,
              lng: displayedTable.value[0].long,
            };
          }
        } else {
          displayedTable.value = table.value.slice(0, 10);
          tarLoc.value = {
            lat: displayedTable.value[0].lat,
            lng: displayedTable.value[0].long,
          };
        }
      }
    };

    const clickCallback = (pageNum) => {
      const startIndex = (pageNum - 1) * itemsPerPage;
      const endIndex = Math.min(startIndex + itemsPerPage, table.value.length);
      displayedTable.value = table.value.slice(startIndex, endIndex);
      tarLoc.value = {
        lat: displayedTable.value[0].lat,
        lng: displayedTable.value[0].long,
      };
    };

    return {
      curTime,
      curLoc,
      tarLoc,
      curInfo,
      targetTime,
      table,
      selectedLoc,
      currentPage,
      itemsPerPage,
      displayedTable,
      getCurrentTime,
      getTargetTime,
      updateCurLoc,
      targetLoc,
      deleteSelected,
      clickCallback,
    };
  },
});
</script>

<style>
.pagination {
  list-style: none;
  display: inline-block;
  padding: 0;
  margin-top: 10px;
}

.pagination li {
  display: inline;
  text-align: center;
}

.pagination a {
  float: left;
  display: block;
  font-size: 14px;
  text-decoration: none;
  padding: 5px 12px;
  color: #fff;
  margin-left: -1px;
  border: 1px solid transparent;
  line-height: 1.5;
  border-color: #ddd;
  color: #4285f4;
  background: #fff;
}

.pagination a.active {
  cursor: default;
}
.pagination a:active {
  background: #eee;
}

.pagination li:first-child a {
  -moz-border-radius: 6px 0 0 6px;
  -webkit-border-radius: 6px;
  border-radius: 6px 0 0 6px;
}
.pagination li:last-child a {
  -moz-border-radius: 0 6px 6px 0;
  -webkit-border-radius: 0;
  border-radius: 0 6px 6px 0;
}

.pagination a.active,
.pagination a:active {
  border-color: #4285f4;
  background: #4285f4;
  color: #fff;
}
</style>
