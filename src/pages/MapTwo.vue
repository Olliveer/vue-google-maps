<script setup lang="ts">
import { GoogleMap, Marker, Polyline } from 'vue3-google-map';
import { generateRandomColor } from '../utils/generateRandomColor';
import data from './data.json';
import VueElementLoading from 'vue-element-loading';
import { reactive, ref, watch } from 'vue';
import Multiselect from '@vueform/multiselect';

const center = {
  lat: -25.43578524053438,
  lng: -49.256734002415286,
};

const googleApiKey = import.meta.env.VITE_GOOGLE_API_MAPS as string;

interface Routes {
  id: string;
}

type Libraries = (
  | 'drawing'
  | 'geometry'
  | 'localContext'
  | 'places'
  | 'visualization'
)[];
const libraries: Libraries = [
  'geometry', // Usado para fazer decode dos polylines
];
const routes = data.map((route) => ({
  ...route,
  color: generateRandomColor(),
}));

const isLoading = ref(false);
const mapRef = ref<typeof GoogleMap>();
const routesValues = reactive({
  value: [] as Routes[],
  options: routes.map((route) => {
    return {
      id: String(route.routeId),
    };
  }),
});

const filteredRoutes = reactive({
  data: routes,
});

watch(
  () => mapRef.value?.ready,
  (ready) => {
    if (!ready) {
      isLoading.value = true;
    } else {
      isLoading.value = false;
    }
    // console.log(mapRef.value, ready);

    // do something with the api using `mapRef.value.api`
    // or with the map instance using `mapRef.value.map`
  }
);

async function handleAddRoute() {
  filteredRoutes.data = routes.filter(({ routeId }) =>
    routesValues.value.some((route) => route.id === String(routeId))
  );
}

function handleRemove() {
  filteredRoutes.data = routes.filter(({ routeId }) =>
    routesValues.value.some((route) => route.id === String(routeId))
  );
}

function handleClearRoutes() {
  filteredRoutes.data = [];
}
</script>

<template>
  <section class="flex-container rounded bg-white shadow m-3">
    <VueElementLoading
      :active="isLoading"
      spinner="spinner"
      color="#FF6700"
      is-full-screen
    />

    <div class="d-flex my-2 p-1">
      <Multiselect
        v-model="routesValues.value"
        @select="handleAddRoute"
        @deselect="handleRemove"
        mode="tags"
        :close-on-select="false"
        :searchable="false"
        :create-option="false"
        :options="routesValues.options"
        value-prop="id"
        label="id"
        :object="true"
        @clear="handleClearRoutes"
      />
    </div>

    <GoogleMap
      ref="mapRef"
      :api-key="googleApiKey"
      style="width: 100%; height: 100vh"
      :center="center"
      :zoom="12"
      :libraries="libraries"
    >
      <template #default="{ ready, api, map, mapTilesLoaded }">
        <!-- First pattern: Here you have access to the API and map instance.
      "ready" is a boolean that indicates when the Google Maps script
      has been loaded and the api and map instance are ready to be used -->
        <template
          v-if="ready"
          v-for="route in filteredRoutes.data"
          :key="route.routeId"
        >
          <Marker
            :key="index"
            v-for="(stop, index) in route.stops"
            :options="{
              position: {
                lat: stop.address.latitude,
                lng: stop.address.longitude,
              },
            }"
          />

          <template v-for="leg in route.legs">
            <Polyline
              v-for="step in leg.steps"
              :key="step.polyline.points"
              :options="{
                path: api.geometry.encoding.decodePath(step.polyline.points),
                geodesic: true,
                strokeColor: route.color,
                strokeOpacity: 1.0,
                strokeWeight: 3,
              }"
            />
          </template>
        </template>
      </template>
    </GoogleMap>
    <!-- <select
      @change="handleChangeRoute"
      style="position: fixed; top: 10%; left: 1%"
      multiple
    >
      <option
        v-for="route in routes"
        :key="route.routeId"
        :value="route.routeId"
      >
        {{ route.routeId }}
      </option>
    </select> -->
  </section>
</template>

<style src="@vueform/multiselect/themes/default.css"></style>
