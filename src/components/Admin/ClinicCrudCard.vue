<template>
  <CardView v-loading="loading" :cardsprops="clinics.map((item) => { return { ...item, cardType: 'admin' } })"
    :myCard="DepartCard" @handleclick="handleclick" style="text-align: center; height: 100%; min-height: 72vh;">
  </CardView>
</template>

<script lang="ts" setup>
import { useHospitalStore } from '@/stores/hospitalData';
import DepartCard from '@/components/DepartCard.vue';
import CardView from '@/views/CardView.vue'
import { type clinic } from '@/api/patient/registrations';
import { onMounted, ref } from 'vue';
import { useRouter, useRoute } from 'vue-router'

const hospitalStore = useHospitalStore();
const loading = ref(true);
const router = useRouter();
const route = useRoute();
const clinics = hospitalStore.clinics;

const handleclick = (clinic: clinic) => {
  router.push({
    name: "crudClinicDoctor",
    params: {
      department: route.query.departmentName as string,
      clinic: clinic.name,
    },
    query: {
      departmentId: route.query.departmentId,
      clinicId: clinic.id,
    }
  })
}

onMounted(async () => {
  await hospitalStore.getClinics(Number(route.query.departmentId))
  loading.value = false;

})
</script>

<style lang="scss" scoped>
.medical-ap {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}
</style>
