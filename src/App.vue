<script setup lang="ts">
import AppTheme from './components/AppTheme.vue'
import { cloneDeep } from 'lodash-es';
import { ref, reactive, } from 'vue'
import type { UnwrapRef } from 'vue';
import { theme } from 'ant-design-vue';
import { PlusOutlined } from "@ant-design/icons-vue";
export interface DataType {
  key: string;
  name: string;
  completed: boolean;
}
export type DataIndex = 'key' | "name" | "completed"
const genId = () => Math.random().toString(36).substring(2, 9);
const initialTheme =
  localStorage.getItem("theme") ??
  (globalThis.matchMedia("(prefers-color-scheme: dark)").matches
    ? "dark"
    : "light");
const currentTheme = ref(initialTheme)
const name = ref('')
const data = reactive(localStorage.getItem("tasks")
  ? (JSON.parse(localStorage.getItem("tasks") as string) as DataType[])
  : [])
function toggleTheme() {
  currentTheme.value = currentTheme.value === 'dark' ? 'light' : 'dark'
}
function addTask() {
  data.push({
    key: genId(),
    name: name.value,
    completed: false,
  })
  name.value = ''
  localStorage.setItem("tasks", JSON.stringify(data));
}
const remove = (record: Partial<DataType>) => {
  const index = data.findIndex((item) => record.key === item.key);
  data.splice(index, 1);
  localStorage.setItem("tasks", JSON.stringify(data));
};

const columns = [
  {
    title: "Name",
    dataIndex: "name",
    width: "30%",
    editable: true,
    sorter: (a: DataType, b: DataType) => {
      const nameA = a.name.toUpperCase(); // ignore upper and lowercase
      const nameB = b.name.toUpperCase(); // ignore upper and lowercase
      if (nameA < nameB) {
        return -1;
      }
      if (nameA > nameB) {
        return 1;
      }

      // names must be equal
      return 0;
    },
    showSorterTooltip: false,
  },
  {
    title: "Completed",
    dataIndex: "completed",
    width: "30%",
    editable: true,
    filters: [
      {
        text: "Completed",
        value: true,
      },
      {
        text: "Pending",
        value: false,
      },
    ],
    onFilter: (value: boolean, record: DataType) =>
      record.completed === value ? true : false,
  },
  {
    title: "Action",
    dataIndex: "action",
    width: "40%",
  },
];
const editableData: UnwrapRef<Record<string, DataType>> = reactive({});
const edit = (key: string) => {
  editableData[key] = cloneDeep(data.filter(item => key === item.key)[0]);
};
const save = (key: string) => {
  Object.assign(data.filter(item => key === item.key)[0], editableData[key]);
  delete editableData[key];
  localStorage.setItem("tasks", JSON.stringify(data));

};
const cancel = (key: string) => {
  delete editableData[key];
};
</script>

<template>
  <a-config-provider :theme="{
    algorithm: currentTheme === 'dark' ? theme.darkAlgorithm : theme.defaultAlgorithm,
  }">
    <div className="container m-2 md:mx-auto text-center w-[95%] h-screen">
      <header className="flex justify-between items-center text-xl font-semibold h-20 dark:text-neutral-100">
        <div className="flex m-4 items-center space-x-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
            <mask id="lineMdCheckAll0">
              <g fill="none" stroke="#fff" stroke-dasharray="24" stroke-dashoffset="24" stroke-linecap="round"
                stroke-linejoin="round" stroke-width="2">
                <path d="M2 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" dur="0.4s" values="24;0" />
                </path>
                <path stroke="#000" strokeWidth={6} d="M7.5 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.4s" dur="0.4s" values="24;0" />
                </path>
                <path d="M7.5 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.4s" dur="0.4s" values="24;0" />
                </path>
              </g>
            </mask>
            <rect width="24" height="24" fill="currentColor" mask="url(#lineMdCheckAll0)" />
          </svg>
          <h1>Vue Todo App</h1>
        </div>
        <AppTheme :theme="currentTheme" @toggle-theme="toggleTheme" />
      </header>
      <main>
        <div className="flex flex-row items-center mb-4">

          <a-input placeholder="Task" allowClear size="large" v-model:value="name" />
          <a-button type="default" size="large" class="!flex items-center ml-4" @click="addTask"
            :disabled="name.trim() === ''">
            <template #icon>
              <PlusOutlined />
            </template>Add Task
          </a-button>
        </div>
        <a-table :dataSource="data" :columns="columns">
          <template #bodyCell="{ column, text, record }">
            <template v-if="['name'].includes(column.dataIndex)">
              <div>
                <a-input v-if="editableData[record.key]"
                  v-model:value="editableData[record.key][column.dataIndex as DataIndex]" style="margin: -5px 0" />
                <template v-else>
                  {{ text }}
                </template>
              </div>
            </template>
            <template v-if="['completed'].includes(column.dataIndex)">
              <div>
                <a-checkbox v-if="editableData[record.key]"
                  v-model:checked="editableData[record.key][column.dataIndex as DataIndex]" />
                <template v-else>
                  <a-checkbox :checked="record['completed']" disabled />
                </template>
              </div>
            </template>
            <template v-else-if="column.dataIndex === 'action'">
              <div class="editable-row-operations">
                <span v-if="editableData[record.key]">
                  <a-space size="middle">
                    <a-typography-link @click="save(record.key)">Save</a-typography-link>
                    <a-popconfirm title="Sure to cancel?" @confirm="cancel(record.key)">
                      <a>Cancel</a>
                    </a-popconfirm>
                  </a-space>
                </span>
                <span v-else>
                  <a-space size="middle">
                    <a-typography-link @click="edit(record.key)">Edit</a-typography-link>
                    <a-popconfirm title="Delete the task" description="Are you sure to delete this task?"
                      @confirm="remove(record.key)" ok-text="Yes" cancel-text="No">
                      <a-typography-link type="danger">Delete</a-typography-link>
                    </a-popconfirm>
                  </a-space>
                </span>
              </div>
            </template>
          </template>
        </a-table>
      </main>
    </div>
  </a-config-provider>
</template>

<style scoped></style>
