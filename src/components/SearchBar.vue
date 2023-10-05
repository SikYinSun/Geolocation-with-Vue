<template>
  <div class="mb-8 w-full rounded-lg bg-gray-200 p-5 w-2/4">
    <div class="flex">
      <div
        class="flex w-10 items-center justify-center rounded-tl-lg rounded-bl-lg border-r border-gray-200 bg-white p-5"
      >
        <svg
          viewBox="0 0 20 20"
          aria-hidden="true"
          class="pointer-events-none absolute w-5 fill-gray-500 transition"
        >
          <path
            d="M16.72 17.78a.75.75 0 1 0 1.06-1.06l-1.06 1.06ZM9 14.5A5.5 5.5 0 0 1 3.5 9H2a7 7 0 0 0 7 7v-1.5ZM3.5 9A5.5 5.5 0 0 1 9 3.5V2a7 7 0 0 0-7 7h1.5ZM9 3.5A5.5 5.5 0 0 1 14.5 9H16a7 7 0 0 0-7-7v1.5Zm3.89 10.45 3.83 3.83 1.06-1.06-3.83-3.83-1.06 1.06ZM14.5 9a5.48 5.48 0 0 1-1.61 3.89l1.06 1.06A6.98 6.98 0 0 0 16 9h-1.5Zm-1.61 3.89A5.48 5.48 0 0 1 9 14.5V16a6.98 6.98 0 0 0 4.95-2.05l-1.06-1.06Z"
          ></path>
        </svg>
      </div>
      <input
        type="text"
        @keyup.enter="handleEnter"
        ref="autocomplete"
        class="w-full bg-white pl-2 text-base font-semibold outline-0"
        placeholder="Enter address"
      />
      <input
      type="button"
      @click="getInformation"
        value="Search"
        class="bg-blue-500 p-2 rounded-tr-lg rounded-br-lg text-white font-semibold hover:bg-blue-800 transition-colors"
      />
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";
import axios from "axios";

export default {
  data() {
    const placeName = ref("");
    const lat = ref("");
    const long = ref("");
    let targetDate = new Date();
    const timestamp = ref("");
    const timeID = ref("");
    const timezone = ref("");
    const tempTime = ref(null);
    let autocomplete;
    let enterKey = false;
    let placeChangedListener = null;

    const handleEnter = () => {
      if (enterKey == false) {
        enterKey = true;
        getInformation();
      }
    };

    const getInformation = () => {
      if (placeChangedListener) {
        google.maps.event.removeListener(placeChangedListener);
      }

      placeChangedListener = this.autocomplete.addListener(
        "place_changed",
        async () => {
          const place = this.autocomplete.getPlace();
          this.placeName = place.name;
          this.lat = place.geometry.location.lat();
          this.long = place.geometry.location.lng();
          this.timestamp =
            targetDate.getTime() / 1000 + targetDate.getTimezoneOffset() * 60;
          this.timezone = null;

          try {
            const response = await axios.get(
              `https://maps.googleapis.com/maps/api/timezone/json?location=${this.lat},${this.long}&timestamp=${this.timestamp}&key=AIzaSyCDKfdwkW6ZAGv6eU5ig7IST4e6612onOM`
            );

            timezone.value = response.data.timeZoneName;
            timeID.value = response.data.timeZoneId;
            tempTime.value =
              response.data.rawOffset * 1000 +
              response.data.dstOffset * 1000 +
              this.timestamp * 1000;

            this.$emit("targetUpdate", {
              placeName: this.placeName,
              lat: this.lat,
              long: this.long,
              time: this.time,
              date: this.date,
              timeID: this.timeID,
              timezone: this.timezone,
            });

            this.$refs.autocomplete.value = "";
            enterKey = false;
          } catch (error) {
            console.error("Error fetching timezone information:", error);
          }
        }
      );
    };

    const date = computed(() => {
      return new Date(tempTime.value).toLocaleDateString("en-US");
    });

    const time = computed(() => {
      return new Date(tempTime.value).toLocaleTimeString("en-US");
    });

    return {
      placeName,
      lat,
      long,
      date,
      time,
      timeID,
      timezone,
      handleEnter,
      getInformation,
    };
  },
  mounted() {
    this.autocomplete = new google.maps.places.Autocomplete(
      this.$refs.autocomplete,
      { types: ["geocode"] }
    );
  },
};
</script>
