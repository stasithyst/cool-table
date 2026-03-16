<template>
  <AFlex class="head">
    <AFlex class="head-search-block">
      <AInputSearch
        v-model:value="filters.search"
        size="large"
        placeholder="Search"
        style="width: 320px"
      />
      <AButton size="large" @click="showFilter = !showFilter"> Filter </AButton>
    </AFlex>

    <AFlex class="head-filter-block">
      <transition name="fade">
        <AFlex class="wrapper" v-if="showFilter">
          <AInput
            v-model:value="filters.firstName"
            placeholder="First Name"
            style="width: 140px"
          />

          <AInput
            v-model:value="filters.lastName"
            placeholder="Last Name"
            style="width: 140px"
          />

          <AInput
            v-model:value="filters.age"
            placeholder="Age"
            style="width: 140px"
          />

          <AInput
            v-model:value="filters.address"
            placeholder="Address"
            style="width: 140px"
          />

          <ASelect
            v-model:value="filters.role"
            allow-clear
            placeholder="Role"
            style="width: 140px"
          >
            <ASelectOption value="admin">Admin</ASelectOption>
            <ASelectOption value="moderator">Moderator</ASelectOption>
            <ASelectOption value="user">User</ASelectOption>
          </ASelect>
          <AButton @click="resetFilters" shape="circle" size="middle">
            <CloseOutlined class="icon" />
          </AButton>
        </AFlex>
      </transition>
    </AFlex>

    <AFlex class="head-button-block">
      <AButton @click="editMode = !editMode">
        {{ editMode ? "Save" : "Edit" }}
      </AButton>
      <AButton @click="open = !open"> Complete Data </AButton>
    </AFlex>

    <AModal v-model:open="open" :style="{ width: '800px' }">
      <ACard
        v-for="users in dataTable"
        :key="users.id"
        :title="`${users.firstName + ' ' + users.lastName}`"
        :style="{ marginTop: '32px' }"
      >
        <AFlex v-for="(value, key) in flatObject(users)" :key="key">
          <p>{{ key }}: {{ value }}</p>
        </AFlex>
      </ACard>
    </AModal>
  </AFlex>

  <ATable
    size="small"
    rowKey="id"
    :dataSource="filteredData"
    :columns="visibleColumns"
    @resizeColumn="handleResizeColumn"
    :loading="loading"
    :error="error"
    :scroll="{ x: 'max-content' }"
    :show-sorter-tooltip="false"
    class="table"
  >
    <template #headerCell="{ column }">
      <div
        v-if="editMode"
        class="table-header"
        draggable="true"
        @dragstart="onDragStart(column)"
        @dragover.prevent
        @drop="onDropList(column)"
      >
        <AFlex class="table-header-cell">
          <ACheckbox v-model:checked="column.visible" @click.stop></ACheckbox>

          <AInput
            v-model:value="column.title"
            placeholder="Title"
            :style="{ minWidth: '120px' }"
          ></AInput>

          <AButton @click="addCol(column)" size="small" class="table-button">
            <PlusOutlined />
          </AButton>
          <AButton
            @click="deleteCol(column.key)"
            size="small"
            class="table-button"
          >
            <DeleteOutlined />
          </AButton>
        </AFlex>
      </div>
      <AFlex v-else>
        {{ column.title }}
      </AFlex>
    </template>

    <template #bodyCell="{ column, record }">
      <AFlex :class="{ disabled: !column.visible }">
        <template v-if="column.key === 'image'">
          <img
            :src="record.image"
            style="width: 50px; height: 50px; object-fit: cover"
            alt="avatar"
          />
        </template>

        <template v-else-if="editMode && column.key === 'id'">
          <AFlex class="first-col">
            <AInput v-model:value="record[column.key]"></AInput>
            <AButton @click="addRow(record)" size="small" class="table-button">
              <PlusOutlined />
            </AButton>
            <AButton
              @click="deleteRow(record.id)"
              size="small"
              class="table-button"
            >
              <DeleteOutlined />
            </AButton>
          </AFlex>
        </template>

        <template v-else-if="editMode && column.key === 'birthDate'">
          <ADatePicker
            v-model:value="record[column.key]"
            format="YYYY-M-D"
            valueFormat="YYYY-M-D"
            style="width: 100%"
          />
        </template>

        <template v-else-if="editMode && column.key === 'gender'">
          <ASelect v-model:value="record[column.key]" style="width: 100%">
            <ASelectOption value="male">male</ASelectOption>
            <ASelectOption value="female">female</ASelectOption>
          </ASelect>
        </template>

        <template v-else-if="editMode">
          <AInput v-model:value="record[column.key]"></AInput>
        </template>

        <template v-else-if="column.key === 'role'">
          <span>
            <ATag
              :key="record.role"
              :color="record.role === 'admin' ? 'pink' : 'green'"
            >
              {{ record.role }}
            </ATag>
          </span>
        </template>

        <template v-else>
          {{ record[column.key] }}
        </template>
      </AFlex>
    </template>
  </ATable>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref, watch, computed } from "vue";
import {
  CloseOutlined,
  DeleteOutlined,
  PlusOutlined,
} from "@ant-design/icons-vue";

const open = ref(false);

const loading = ref(false);
const error = ref(null);

const showFilter = ref(false);

const dataTable = ref([]);

const editMode = ref(false);

const filters = reactive({
  search: "",
  address: "",
  firstName: "",
  lastName: "",
  age: "",
  role: undefined,
});

const fetchData = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await fetch("https://dummyjson.com/users");

    if (!response.ok) {
      throw new Error("Ошибка запроса");
    }

    const data = await response.json();
    dataTable.value = data.users;
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchData();
});

const flatData = computed(() => {
  return dataTable.value.map((user) => ({
    ...user,
    "hair.color": user.hair?.color,
    "hair.type": user.hair?.type,
    "address.address": user.address?.address,
    "address.city": user.address?.city,
    "address.state": user.address?.state,
    "address.postalCode": user.address?.postalCode,
    "address.country": user.address?.country,
    "company.name": user.company?.name,
    "company.department": user.company?.department,
    "company.title": user.company?.title,
  }));
});

const flatObject = (obj) => {
  function flat(o) {
    return Object.entries(o).flatMap(([key, val]) => {
      if (typeof val === "object") return flat(val);

      return [[key, val]];
    });
  }

  return Object.fromEntries(flat(obj));
};

const filteredData = computed(() => {
  return flatData.value.filter((user) => {
    if (filters.search) {
      const searchValue = filters.search.toLowerCase();
      const userData = JSON.stringify(user).toLowerCase();
      if (!userData.includes(searchValue)) {
        return false;
      }
    }

    if (
      filters.firstName &&
      !user.firstName.toLowerCase().includes(filters.firstName.toLowerCase())
    ) {
      return false;
    }

    if (
      filters.lastName &&
      !user.lastName.toLowerCase().includes(filters.lastName.toLowerCase())
    ) {
      return false;
    }

    if (filters.age && String(user.age) !== String(filters.age)) {
      return false;
    }

    if (filters.address) {
      const place = JSON.stringify(user.address).toLowerCase();
      if (!place.includes(filters.address.toLowerCase())) {
        return false;
      }
    }

    return !(filters.role && user.role !== filters.role);
  });
});

const resetFilters = () => {
  filters.search = "";
  filters.address = "";
  filters.firstName = "";
  filters.lastName = "";
  filters.age = "";
  filters.role = undefined;
};

const columns = ref([
  {
    title: "ID",
    dataIndex: "id",
    key: "id",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.id - b.id,
  },
  {
    title: "First Name",
    dataIndex: "firstName",
    key: "firstName",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.firstName.localeCompare(b.firstName),
  },
  {
    title: "Last Name",
    dataIndex: "lastName",
    key: "lastName",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.lastName.localeCompare(b.lastName),
  },
  {
    title: "Maiden Name",
    dataIndex: "maidenName",
    key: "maidenName",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.maidenName.localeCompare(b.maidenName),
  },
  {
    title: "Age",
    dataIndex: "age",
    key: "age",
    resizable: true,
    minWidth: 60,
    visible: true,
    sorter: (a, b) => a.age - b.age,
  },
  {
    title: "Gender",
    dataIndex: "gender",
    key: "gender",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.gender.localeCompare(b.gender),
  },
  {
    title: "Email",
    dataIndex: "email",
    key: "email",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.email.localeCompare(b.email),
  },
  {
    title: "Phone",
    dataIndex: "phone",
    key: "phone",
    resizable: true,
    minWidth: 150,
    visible: true,
    sorter: (a, b) => a.phone.localeCompare(b.phone),
  },
  {
    title: "Username",
    dataIndex: "username",
    key: "username",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.username.localeCompare(b.username),
  },
  {
    title: "Password",
    dataIndex: "password",
    key: "password",
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.password.localeCompare(b.password),
  },
  {
    title: "Birth Date",
    dataIndex: "birthDate",
    key: "birthDate",
    resizable: true,
    minWidth: 100,
    visible: true,
    sorter: (a, b) => a.birthDate.localeCompare(b.birthDate),
  },
  {
    title: "Image",
    dataIndex: "image",
    resizable: true,
    key: "image",
    visible: true,
    minWidth: 80,
  },
  {
    title: "Blood Group",
    dataIndex: "bloodGroup",
    key: "bloodGroup",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.bloodGroup.localeCompare(b.bloodGroup),
  },
  {
    title: "Height",
    dataIndex: "height",
    key: "height",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.height - b.height,
  },
  {
    title: "Weight",
    dataIndex: "weight",
    key: "weight",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.weight - b.weight,
  },
  {
    title: "Eye Color",
    dataIndex: "eyeColor",
    key: "eyeColor",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.eyeColor.localeCompare(b.eyeColor),
  },
  {
    title: "Hair Color",
    dataIndex: ["hair", "color"],
    key: "hair.color",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.hair.color.localeCompare(b.hair.color),
  },
  {
    title: "Hair Type",
    dataIndex: ["hair", "type"],
    key: "hair.type",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.hair.type.localeCompare(b.hair.type),
  },
  {
    title: "IP",
    dataIndex: "ip",
    key: "ip",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.ip.localeCompare(b.ip),
  },
  {
    title: "Address",
    dataIndex: ["address", "address"],
    key: "address.address",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.address.address.localeCompare(b.address.address),
  },
  {
    title: "City",
    dataIndex: ["address", "city"],
    key: "address.city",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.address.city.localeCompare(b.address.city),
  },
  {
    title: "State",
    dataIndex: ["address", "state"],
    key: "address.state",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.address.state.localeCompare(b.address.state),
  },
  {
    title: "Postal Code",
    dataIndex: ["address", "postalCode"],
    key: "address.postalCode",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.address.postalCode.localeCompare(b.address.postalCode),
  },
  {
    title: "Country",
    dataIndex: ["address", "country"],
    key: "address.country",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.address.country.localeCompare(b.address.country),
  },
  {
    title: "University",
    dataIndex: "university",
    key: "university",
    resizable: true,
    minWidth: 100,
    visible: true,
    sorter: (a, b) => a.university.localeCompare(b.university),
  },
  {
    title: "Company Name",
    dataIndex: ["company", "name"],
    key: "company.name",
    resizable: true,
    minWidth: 100,
    visible: true,
    sorter: (a, b) => a.company.name.localeCompare(b.company.name),
  },
  {
    title: "Company Department",
    dataIndex: ["company", "department"],
    key: "company.department",
    resizable: true,
    minWidth: 80,
    visible: true,
    sorter: (a, b) => a.company.department.localeCompare(b.company.department),
  },
  {
    title: "Company Title",
    dataIndex: ["company", "title"],
    key: "company.title",
    resizable: true,
    width: 100,
    visible: true,
    sorter: (a, b) => a.company.title.localeCompare(b.company.title),
  },
  {
    title: "Role",
    dataIndex: "role",
    key: "role",
    resizable: true,
    width: 80,
    visible: true,
    sorter: (a, b) => a.role.localeCompare(b.role),
  },
]);

function handleResizeColumn(w, col) {
  col.width = w;
}

const visibleColumns = computed(() => {
  if (editMode.value) {
    return columns.value;
  }
  return columns.value.filter((col) => col.visible);
});

const dragItem = ref(null);

function onDragStart(col: any) {
  dragItem.value = col;
}

function onDropList(targetCol: any) {
  if (!dragItem.value) return;

  const firstIndex = columns.value.findIndex(
    (t) => t.key === dragItem.value.key,
  );

  const secondIndex = columns.value.findIndex((t) => t.key === targetCol.key);

  if (firstIndex === -1 || secondIndex === -1) {
    return;
  }

  const moved = columns.value.splice(firstIndex, 1)[0];
  columns.value.splice(secondIndex, 0, moved);

  dragItem.value = null;
}

function addCol(targetCol: any) {
  const key = "" + columns.value.length + 1;
  const newCol = {
    title: "",
    dataIndex: key,
    key: key,
    resizable: true,
    width: 80,
    visible: true,
    sorter: (a, b) => a[key].localeCompare(b[key]),
  };

  const index = columns.value.findIndex((col) => col.key === targetCol.key);

  columns.value.splice(index + 1, 0, newCol);
}

function addRow(targetRow: any) {
  const index = dataTable.value.findIndex((row) => row.id === targetRow.id);

  for (let i = index + 1; i < dataTable.value.length; i++) {
    dataTable.value[i].id++;
  }

  dataTable.value.splice(index + 1, 0, { id: targetRow.id + 1 });
}

function deleteRow(rowId: any) {
  dataTable.value = dataTable.value.filter((row) => row.id !== rowId);
}

function deleteCol(colKey: any) {
  columns.value = columns.value.filter((col) => col.key !== colKey);
}

watch(
  columns,
  (val) => {
    localStorage.setItem("table", JSON.stringify(val));
  },
  { deep: true },
);

onMounted(() => {
  const saved = localStorage.getItem("table");
  if (saved) {
    columns.value = JSON.parse(saved);
  }
});
</script>

<style scoped lang="scss">
.table,
.head {
  padding: 1rem;
}

.head {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  align-items: center;
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.head-search-block,
.head-filter-block,
.head-button-block {
  display: flex;
  gap: 1rem;
}

:deep(.ant-select-selector) {
  display: flex;
  align-items: center;
}

:deep(.ant-table) {
  table-layout: fixed;
}

:deep(.ant-tooltip .ant-tooltip-inner) {
  display: none;
}

:deep(.ant-table-tbody > tr > td) {
  height: 60px;
}

.icon {
  color: #d9d9d9;
}

:deep(.ant-btn:hover .icon) {
  color: #4096ff;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.wrapper {
  display: flex;
  gap: 1rem;
}

.disabled {
  opacity: 0.35;
}

.table-header-cell,
.first-col {
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>
