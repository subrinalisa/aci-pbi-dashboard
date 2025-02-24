<script setup>
import Spinner from "../../components/Spinner.vue";
import { ref, onMounted } from "vue";
import axios from "axios";
import MainLayout from "@/components/MainLayout.vue";
import { apiBase } from "@/config";
import { showNotification } from "@/utilities/notification";
import { useRoute } from "vue-router";
import router from "@/router";
import Cookies from "js-cookie";

const config = {
  headers: {
    Authorization: `Bearer ${Cookies.get("token")}`,
  },
};

const form = ref({
  report_type: "",
  name: "",
  description: "",
  url: "",
  module: "",
  report_id: "",
  secondary_url: "",
});
const isLoading = ref(false);
const isFetching = ref(false);

const editNew = async (data, router) => {
  isLoading.value = true;
  try {
    const res = await axios.put(`${apiBase}/permissions/${id}`, data, config);
    isLoading.value = false;

    if (res?.data?.status == "success") {
      console.log(res?.data?.auth_user_permission?.all_permissions);
      showNotification(res?.data?.status, res?.data?.message);
      localStorage.setItem(
        "all_permissions",
        JSON.stringify(res?.data?.auth_user_permission?.all_permissions)
      );
      router.go(-1);
    }
  } catch (err) {
    isLoading.value = false;
    console.log(err);
  }
};
const route = useRoute();
const { id } = route?.params;

const getData = async () => {
  isFetching.value = true;
  try {
    const res = await axios.get(`${apiBase}/permissions/${id}/edit`, config);
    isFetching.value = false;

    if (res?.data?.status == "success") {
      // console.log(res?.data?.permission);
      const result = res?.data?.permission;
      form.value = {
        report_type: result?.report_type,
        name: result?.name,
        description: result?.description,
        url: result?.url,
        module: result?.module,
        report_id: result?.report_id,
        secondary_url: result?.secondary_url,
      };
    }
  } catch (err) {
    isFetching.value = false;
    console.log(err);
  }
};
const resetForm = () => {
  form.value = {
    report_type: "",
    name: "",
    description: "",
    url: "",
    module: "",
    report_id: "",
    secondary_url: "",
  };
};
onMounted(async () => {
  await getData();
});
</script>

<template>
  <MainLayout>
    <a-skeleton v-if="isFetching" />
    <div v-else>
      <div class="md:flex justify-between items-center">
        <h2 class="title mb-3">Update Dashboard</h2>
        <button type="button" class="c-btn mb-3" @click="$router.go(-1)">Back</button>
      </div>
      <form @submit.prevent="editNew(form, $router)" class="space-y-3">
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">Report Type</label>
          <select class="border w-full rounded px-3 py-1 outline-none" v-model="form.report_type">
            <option value=""></option>
            <option value="PowerBI Report">PowerBI Report</option>
            <option value="PowerBI Dashboard">PowerBI Dashboard</option>
            <option value="PowerBI Tile">PowerBI Tile</option>
            <option value="PowerBI URL">PowerBI URL</option>
            <option value="Non Power BI">Non Power BI</option>
          </select>
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2"
            >Title <span class="text-red-600">*</span></label
          >
          <input
            id="name"
            type="text"
            placeholder="Title"
            class="border w-full rounded px-3 py-1 outline-none"
            required
            v-model="form.name"
          />
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">Description</label>
          <input
            id="name"
            type="text"
            placeholder="Description"
            class="border w-full rounded px-3 py-1 outline-none"
            v-model="form.description"
          />
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">Url</label>
          <input
            id="name"
            type="text"
            placeholder="Url"
            class="border w-full rounded px-3 py-1 outline-none"
            v-model="form.url"
          />
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">GroupID </label>
          <input
            id="name"
            type="text"
            placeholder="GroupID"
            class="border w-full rounded px-3 py-1 outline-none"
            v-model="form.module"
          />
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">ReportID</label>
          <input
            id="name"
            type="text"
            placeholder="ReportID"
            class="border w-full rounded px-3 py-1 outline-none"
            v-model="form.report_id"
          />
        </div>
        <div class="md:flex justify-between items-center">
          <label for="name" class="text-sm w-44 mb-2">Secondary Url:</label>
          <input
            id="name"
            type="text"
            placeholder="Secondary Url:"
            class="border w-full rounded px-3 py-1 outline-none"
            v-model="form.secondary_url"
          />
        </div>

        <div class="flex gap-x-3 mt-5">
          <button type="submit" class="c-btn">Update <Spinner v-if="isLoading" /></button>
          <button type="button" @click="$router.go(-1)" class="d-btn">Cancel</button>
        </div>
      </form>
    </div>
  </MainLayout>
</template>

<style scoped>
.scrollable-content {
  overflow-y: auto;
}
</style>
