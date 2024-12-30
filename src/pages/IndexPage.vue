<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <q-form ref="formRef" class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名"
                 :rules="[val => !!val || '姓名不可為空']"
                 lazy-rules/>
        <q-input v-model="tempData.age" label="年齡"
                 type="number"
                 :rules="[val => !!val || '年齡不可為空', val => val > 0 || '年齡必須大於 0']"
                 lazy-rules/>
        <q-btn color="primary" class="q-mt-md"
               :label="isEditing ? '修改' : '新增'"
               @click="addData(tempData.name, tempData.age)"></q-btn>
      </q-form>

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
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
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
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
    id: 0,
  },
]);
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

const tempData = ref({
  name: '',
  age: '',
});

const formRef = ref(null);
const isEditing = ref(false);

async function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    isEditing.value = true;
    tempData.value.name = data.name;
    tempData.value.age = data.age;
  } else if (btn.status === 'delete') {
    if (confirm(`確定要刪除 ${data.name} 嗎？`)) {
      //blockData.value = blockData.value.filter((row) => row.id !== data.id);
      try {
        const response = await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`);

        if (response.status === 204) {
          blockData.value = blockData.value.filter((row) => row.id !== data.id);
          alert('刪除成功');
        } else {
          alert('刪除失敗，請稍後再試');
        }
      } catch (error) {
        console.error('刪除失敗:', error);
        alert('刪除失敗，請稍後重試');
      }
    }
    }
}
async function addData(name: string, age: number) {
  if (!formRef.value) return;

  const isValid = await formRef.value.validate();
  if (!isValid) {
    console.error('表單驗證失敗');
    return;
  }
  try {
    if (isEditing.value) {
      const response = await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
        name,
        age,
      });
      if (response) {
        const updatedRowIndex = blockData.value.findIndex((item) => item.name === tempData.value.name);
        blockData.value[updatedRowIndex] = {
          id: blockData.value[updatedRowIndex].id, // 保持原有 ID
          name,
          age,
        };
      }
      isEditing.value = false;
    }
    else{
      const response = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
        name,
        age: Number(age),
      });

      if(response)
      {
        blockData.value = [
          ...blockData.value,
          {
            id:  blockData.value.length + 1,
            name: tempData.value.name,
            age: tempData.value.age,
          },
        ];
      }
    }

    tempData.value.name = '';
    tempData.value.age = '';

    formRef.value.resetValidation();
  } catch (error) {
    console.error('新增失敗:', error);
    alert('新增失敗，請稍後重試');
  }
}
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
