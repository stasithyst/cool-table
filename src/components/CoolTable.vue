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
  </AFlex>
  <ATable
    size="small"
    rowKey="id"
    :dataSource="filteredData"
    :columns="columns"
    :loading="loading"
    :error="error"
    :scroll="{ x: 'max-content', y: 800 }"
    class="table"
  >
    <template #bodyCell="{ column, record }">
      <template v-if="column.key === 'image'">
        <img
          :src="record.image"
          style="width: 50px; height: 50px; object-fit: cover"
          alt="avatar"
        />
      </template>
      <template v-if="column.key === 'role'">
        <span>
          <ATag
            :key="record.role"
            :color="record.role === 'admin' ? 'pink' : 'green'"
          >
            {{ record.role }}
          </ATag>
        </span>
      </template>
    </template>
  </ATable>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref, computed } from "vue";
import { CloseOutlined } from "@ant-design/icons-vue";
const loading = ref(false);
const error = ref(null);

const showFilter = ref(false);

const dataTable = ref([]);

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

const filteredData = computed(() => {
  return dataTable.value.filter((user) => {
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

const columns = [
  {
    title: "ID",
    dataIndex: "id",
    key: "id",
    width: 80,
    sorter: (a, b) => a.id - b.id,
  },
  {
    title: "First Name",
    dataIndex: "firstName",
    key: "firstName",
    width: 80,
    sorter: (a, b) => a.firstName.localeCompare(b.firstName),
  },
  {
    title: "Last Name",
    dataIndex: "lastName",
    key: "lastName",
    width: 80,
    sorter: (a, b) => a.lastName.localeCompare(b.lastName),
  },
  {
    title: "Maiden Name",
    dataIndex: "maidenName",
    key: "maidenName",
    width: 80,
    sorter: (a, b) => a.maidenName.localeCompare(b.maidenName),
  },
  {
    title: "Age",
    dataIndex: "age",
    key: "age",
    width: 60,
    sorter: (a, b) => a.age - b.age,
  },
  {
    title: "Gender",
    dataIndex: "gender",
    key: "gender",
    width: 80,
    sorter: (a, b) => a.gender.localeCompare(b.gender),
  },
  {
    title: "Email",
    dataIndex: "email",
    key: "email",
    width: 80,
    sorter: (a, b) => a.email.localeCompare(b.email),
  },
  {
    title: "Phone",
    dataIndex: "phone",
    key: "phone",
    width: 150,
    sorter: (a, b) => a.phone.localeCompare(b.phone),
  },
  {
    title: "Username",
    dataIndex: "username",
    key: "username",
    width: 60,
    sorter: (a, b) => a.username.localeCompare(b.username),
  },
  {
    title: "Password",
    dataIndex: "password",
    key: "password",
    width: 80,
    sorter: (a, b) => a.password.localeCompare(b.password),
  },
  {
    title: "Birth Date",
    dataIndex: "birthDate",
    key: "birthDate",
    width: 100,
    sorter: (a, b) => a.birthDate.localeCompare(b.birthDate),
  },
  {
    title: "Image",
    dataIndex: "image",
    key: "image",
    width: 80,
  },
  {
    title: "Blood Group",
    dataIndex: "bloodGroup",
    key: "bloodGroup",
    width: 80,
    sorter: (a, b) => a.bloodGroup.localeCompare(b.bloodGroup),
  },
  {
    title: "Height",
    dataIndex: "height",
    key: "height",
    width: 80,
    sorter: (a, b) => a.height - b.height,
  },
  {
    title: "Weight",
    dataIndex: "weight",
    key: "weight",
    width: 80,
    sorter: (a, b) => a.weight - b.weight,
  },
  {
    title: "Eye Color",
    dataIndex: "eyeColor",
    key: "eyeColor",
    width: 80,
    sorter: (a, b) => a.eyeColor.localeCompare(b.eyeColor),
  },
  {
    title: "Hair Color",
    dataIndex: ["hair", "color"],
    key: "hair.color",
    width: 80,
    sorter: (a, b) => a.hair.color.localeCompare(b.hair.color),
  },
  {
    title: "Hair Type",
    dataIndex: ["hair", "type"],
    key: "hair.type",
    width: 80,
    sorter: (a, b) => a.hair.type.localeCompare(b.hair.type),
  },
  {
    title: "IP",
    dataIndex: "ip",
    key: "ip",
    width: 80,
    sorter: (a, b) => a.ip.localeCompare(b.ip),
  },
  {
    title: "Address",
    dataIndex: ["address", "address"],
    key: "address.address",
    width: 80,
    sorter: (a, b) => a.address.address.localeCompare(b.address.address),
  },
  {
    title: "City",
    dataIndex: ["address", "city"],
    key: "address.city",
    width: 80,
    sorter: (a, b) => a.address.city.localeCompare(b.address.city),
  },
  {
    title: "State",
    dataIndex: ["address", "state"],
    key: "address.state",
    width: 80,
    sorter: (a, b) => a.address.state.localeCompare(b.address.state),
  },
  {
    title: "Postal Code",
    dataIndex: ["address", "postalCode"],
    key: "address.postalCode",
    width: 80,
    sorter: (a, b) => a.address.postalCode.localeCompare(b.address.postalCode),
  },
  {
    title: "Country",
    dataIndex: ["address", "country"],
    key: "address.country",
    width: 80,
    sorter: (a, b) => a.address.country.localeCompare(b.address.country),
  },
  {
    title: "University",
    dataIndex: "university",
    key: "university",
    width: 100,
    sorter: (a, b) => a.university.localeCompare(b.university),
  },
  {
    title: "Company Name",
    dataIndex: ["company", "name"],
    key: "company.name",
    width: 100,
    sorter: (a, b) => a.company.name.localeCompare(b.company.name),
  },
  {
    title: "Company Department",
    dataIndex: ["company", "department"],
    key: "company.department",
    width: 80,
    sorter: (a, b) => a.company.department.localeCompare(b.company.department),
  },
  {
    title: "Company Title",
    dataIndex: ["company", "title"],
    key: "company.title",
    width: 100,
    sorter: (a, b) => a.company.title.localeCompare(b.company.title),
  },
  {
    title: "Role",
    dataIndex: "role",
    key: "role",
    width: 80,
    sorter: (a, b) => a.role.localeCompare(b.role),
  },
];
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

.head-search-block,
.head-filter-block {
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
  transform: translateY(-3px);
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
</style>
