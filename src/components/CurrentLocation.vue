<template>
  <div class="my-2 mb-8 flex items-center justify-center">
    <div class="flex">
      <button
        @click="getPosition"
        class="before:ease relative h-12 w-40 overflow-hidden border border-black shadow-2xl before:absolute before:left-0 before:-ml-2 before:h-48 before:w-48 before:origin-top-right before:-translate-x-full before:translate-y-12 before:-rotate-90 before:bg-gray-900 before:transition-all before:duration-300 hover:text-white hover:shadow-black hover:before:-rotate-180"
      >
        <span class="relative z-10">Get Current Location</span>
      </button>
    </div>
  </div>
</template>

<script>
import { ref, watch, computed } from "vue";
import axios from "axios";

export default {
  data() {
    const lat = ref(null);
    const long = ref(null);
    const tempTime = ref(null);
    const timeID = ref(null);
    const timezone = ref(null);

    const getPosition = async () => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
          lat.value = position.coords.latitude;
          long.value = position.coords.longitude;
          tempTime.value = position.timestamp;

          try {
            const response = await axios.get(
              `https://maps.googleapis.com/maps/api/timezone/json?location=${
                lat.value
              },${long.value}&timestamp=${
                tempTime.value / 1000
              }&key=AIzaSyCDKfdwkW6ZAGv6eU5ig7IST4e6612onOM`
            );
            timezone.value = response.data.timeZoneName;
            timeID.value = response.data.timeZoneId;
            this.$emit("curUpdate", {
              placeName: "Local",
              lat: this.lat,
              long: this.long,
              date: this.date,
              time: this.time,
              timeID: this.timeID,
              timezone: this.timezone,
            });
          } catch (error) {
            console.error("Error fetching timezone information:", error);
          }
        });
      }
    };

    const date = computed(() => {
      return new Date(tempTime.value).toLocaleDateString("en-US");
    });

    const time = computed(() => {
      return new Date(tempTime.value).toLocaleTimeString("en-US");
    });

    return {
      lat,
      long,
      date,
      time,
      timeID,
      timezone,
      getPosition,
    };
  },
};
</script>
