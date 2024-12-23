<script setup>
import { apiBase, config } from "@/config";
import { onMounted, ref } from "vue";
import MainLayout from "@/components/MainLayout.vue";
import axios from "axios";
import { DeleteOutlined, EditOutlined, PlusOutlined } from "@ant-design/icons-vue";
import { showNotification } from "@/utilities/notification";
import Spinner from "@/components/Spinner.vue";

const isLoading = ref(false);
const allData = ref([]);

// Pagination
const search = ref("");
const paginate = ref(10);
const total = ref(0);

const getData = async (page = "") => {
  isLoading.value = true;
  try {
    const res = await axios.get(
      `${apiBase}/permission_list_with_paginate?search=${search.value}&paginate=${paginate.value}&page=${page}`,
      config
    );

    isLoading.value = false;
    if (res?.data?.status == "success") {
      allData.value = res?.data?.permissions?.data;
      total.value = res?.data?.permissions?.total;
    }
  } catch (err) {
    isLoading.value = false;
  }
};
const isDeleting = ref(false);

const deleteData = async (id) => {
  isDeleting.value = true;
  try {
    const res = await axios.delete(`${apiBase}/permissions/${id}`, config);

    isDeleting.value = false;
    showNotification(res?.data?.status, res?.data?.message);
    if (res?.data?.status == "success") await getData();
  } catch (err) {
    isDeleting.value = false;
  }
};
/* Create */
const open = ref(false);
const form = ref({
  report_type: "",
  name: "",
  description: "",
  url: "",
  module: "",
  report_id: "",
  secondary_url: "",
});
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
const isInserting = ref(false);

const createNew = async (data) => {
  isInserting.value = true;
  try {
    const res = await axios.post(`${apiBase}/permissions`, data, config);
    isInserting.value = false;
    showNotification(res?.data?.status, res?.data?.message);
    if (res?.data?.status == "success") {
      // showNotification(res?.data?.status, res?.data?.message);
      resetForm();
      await getData();
      open.value = false;
    }
  } catch (err) {
    showNotification(res?.data?.status, res?.data?.message);
    isInserting.value = false;
    console.log(err);
  }
};
/* onMounted */
onMounted(async () => {
  await getData();
});
</script>

<template>
  <MainLayout>
    <div class="md:flex justify-between mb-3">
      <!-- Search -->
      <div class="grow mb-3">
        <input
          type="text"
          placeholder="Search..."
          class="search"
          v-model="search"
          @input="getData()"
        />
      </div>
      <div class="mb-3">
        <button @click="open = true" class="c-btn flex items-center gap-x-2">
          <PlusOutlined />
          <span>Add New</span>
        </button>
      </div>
    </div>
    <h2 class="title">Manage Dashboards ({{ total }})</h2>
    <table class="purchase-table mb-3">
      <thead class="table-header">
        <tr>
          <th class="w-16">Actions</th>
          <th>Type</th>
          <th>Title</th>
          <th>Description</th>
          <th>Url</th>
          <th>Group ID</th>
          <th>Report ID</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="!allData?.length && !isLoading">
          <td colspan="8">No Data</td>
        </tr>
        <tr v-if="isLoading">
          <td colspan="8">Loading...</td>
        </tr>

        <tr v-for="item in allData" :key="item?.id">
          <td>
            <div class="flex gap-x-2">
              <button
                type="button"
                class="edit_btn"
                @click="$router.push({ name: 'permission-edit', params: { id: item?.id } })"
              >
                <EditOutlined class="align-middle" />
              </button>

              <a-popconfirm title="Are you sure delete?" @confirm="deleteData(item?.id)">
                <button type="button" class="del_btn">
                  <DeleteOutlined class="align-middle" />
                </button>
              </a-popconfirm>
            </div>
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Type:</span>
            {{ item?.report_type || "-" }}
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Title:</span>
            {{ item?.name || "-" }}
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Description:</span>
            {{ item?.description || "-" }}
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Url:</span>
            <a
              :href="item?.url"
              target="_blank"
              class="text-secondary"
              :class="{ 'text-gray-300': !item?.url }"
              >Open</a
            >
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Group ID:</span>
            {{ item?.module || "-" }}
          </td>
          <td>
            <span class="md:hidden mr-1 font-semibold">Report ID:</span>
            {{ item?.report_id || "-" }}
          </td>
        </tr>
      </tbody>
    </table>
    <!-- Pagination -->
    <a-pagination
      v-if="total > paginate"
      size="small"
      :total="total"
      :show-total="(total) => `Total ${total} items`"
      :show-size-changer="false"
      show-quick-jumper
      @change="(page) => getData(page)"
    />
    <!-- Create New -->
    <a-modal v-model:open="open" title="Create New Dashboard" :footer="false" @cancel="resetForm()">
      <form @submit.prevent="createNew(form)" class="space-y-3">
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
          <label for="name" class="text-sm w-44 mb-2"
            >GroupID <span class="text-red-600">*</span></label
          >
          <input
            id="name"
            type="text"
            placeholder="GroupID"
            class="border w-full rounded px-3 py-1 outline-none"
            required
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

        <button type="submit" class="c-btn">Save <Spinner v-if="isInserting" /></button>
      </form>
    </a-modal>
  </MainLayout>
</template>
