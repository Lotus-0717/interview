<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="submitData">新增</q-btn>
        <q-btn color="primary" class="q-mt-md q-ml-md" @click="searchData"
          >搜尋</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props">
              <div v-if="editingRowId !== props.row.id">{{ col.value }}</div>
              <input v-else v-model="props.row[col.field]" />
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <template v-if="editingRowId === props.row.id">
                <q-btn
                  v-for="btn in checkButtons"
                  :key="btn.label"
                  @click="confirmEdit(props.row)"
                  :icon="btn.icon"
                  size="sm"
                  color="grey-6"
                  round
                  dense
                  class="q-ml-md"
                  padding="5px 5px"
                >
                  <q-tooltip
                    transition-show="scale"
                    transition-hide="scale"
                    anchor="top middle"
                    self="bottom middle"
                    :offset="[10, 10]"
                    >{{ btn.label }}</q-tooltip
                  >
                </q-btn>
              </template>
              <template v-else>
                <q-btn
                  v-for="btn in tableButtons"
                  :key="btn.label"
                  @click="handleClickOption(btn, props.row)"
                  :icon="btn.icon"
                  size="sm"
                  color="grey-6"
                  round
                  dense
                  class="q-ml-md"
                  padding="5px 5px"
                >
                  <q-tooltip
                    transition-show="scale"
                    transition-hide="scale"
                    anchor="top middle"
                    self="bottom middle"
                    :offset="[10, 10]"
                    >{{ btn.label }}</q-tooltip
                  >
                </q-btn>
              </template>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([]);
const originalData = ref([]);
const tempData = ref({
  name: '',
  age: '',
});

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const checkButtons = ref([
  {
    label: '確認',
    icon: 'check',
    status: 'check',
  },
]);
const editingRowId = ref(null);

async function getData() {
  try {
    const response = await axios.get(
      'https://demo.mercuryfire.com.tw:49110/crudTest/a'
    );
    blockData.value = response.data.result;
    originalData.value = response.data.result.slice();
  } catch (error) {
    console.error('失敗:', error);
  }
}

function handleClickOption(btn, rowData) {
  console.log(btn, rowData);
  if (btn.status === 'edit') {
    editRow(rowData);
  } else if (btn.status === 'delete') {
    deleteRow(rowData);
  }
}

function editRow(rowData) {
  editingRowId.value = rowData.id;
}
async function confirmEdit(rowData) {
  const verify = validateNameAndAge(rowData.name, rowData.age);
  if (!verify.isValid) {
    alert(verify.message);
    return;
  }
  try {
    const response = await axios.patch(
      'https://demo.mercuryfire.com.tw:49110/crudTest',
      {
        ID: rowData.id,
        name: rowData.name,
        age: Number(rowData.age),
      }
    );
    console.log('成功:', response.data);
    getData();
    editingRowId.value = null;
  } catch (error) {
    console.error('失敗:', error);
  }
}

async function deleteRow(rowData) {
  try {
    const response = await axios.delete(
      `https://demo.mercuryfire.com.tw:49110/crudTest/${rowData.id}`
    );
    console.log('成功:', response.data);
    tempData.value.name = '';
    tempData.value.age = '';
    getData();
  } catch (error) {
    console.error('失敗:', error);
  }
}

async function submitData() {
  const verify = validateNameAndAge(tempData.value.name, tempData.value.age);
  if (!verify.isValid) {
    alert(verify.message);
    return;
  }
  try {
    const response = await axios.post(
      'https://demo.mercuryfire.com.tw:49110/crudTest',
      {
        name: tempData.value.name,
        age: Number(tempData.value.age),
      }
    );
    console.log('成功:', response.data);
    tempData.value.name = '';
    tempData.value.age = '';
    getData();
  } catch (error) {
    console.error('失敗:', error);
  }
}

function searchData() {
  blockData.value = originalData.value.filter((item) => {
    const matchesName = tempData.value.name
      ? item.name.includes(tempData.value.name)
      : true;
    const matchesAge = tempData.value.age
      ? item.age === Number(tempData.value.age)
      : true;
    return matchesName && matchesAge;
  });
}

function validateNameAndAge(name, age) {
  const nameRegex = /^[a-zA-Z\u4e00-\u9fa5]+$/;
  if (!name || !nameRegex.test(name)) {
    return {
      isValid: false,
      message: '姓名不得為空，且僅限中英文，不包含特殊符號。',
    };
  }

  if (!Number.isInteger(Number(age)) || age < 0) {
    return { isValid: false, message: '年齡格式必須為正整數。' };
  }

  return { isValid: true, message: '驗證成功。' };
}

onMounted(getData);
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
