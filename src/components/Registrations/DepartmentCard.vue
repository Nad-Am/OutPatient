<template>
  <CardView v-loading="loading" @handleclick="handleclick" style="text-align: center; height: 100%; min-height: 72vh;"
    :cardsprops="departs.map((item) => { return { ...item, cardType: 'patient' } })" :myCard="DepartCard">
  </CardView>
</template>

<script lang="ts" setup>
import { useHospitalStore } from '@/stores/hospitalData';
import DepartCard from '@/components/DepartCard.vue';
import CardView from '@/views/CardView.vue'
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router'
import { type department } from '@/api/patient/registrations';

const hospitalStore = useHospitalStore();
const loading = ref(true);
const router = useRouter();
const departs = hospitalStore.departs;

const handleclick = (department: department) => {
  router.push({
    name: "clinic",
    params: {
      department: department.name,
    },
    query: {
      departmentName: department.name,
      departmentId: department.id,
    }
  })
}

onMounted(async () => {
  await hospitalStore.getDepartments();
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
