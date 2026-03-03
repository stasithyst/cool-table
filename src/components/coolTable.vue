<template>
  <AFlex class="head">
    <AInputSearch
        v-model:value="filters.search"
        size="large"
        placeholder="Search"
        style="width: 320px"
    />
    <AButton
        size="large"
        @click="showFilter = !showFilter"
    >
      Filter
    </AButton>
    <AInput
        v-if="showFilter"
        v-model:value="filters.firstName"
        placeholder="First Name"
    />

    <AInput
        v-if="showFilter"
        v-model:value="filters.lastName"
        placeholder="Last Name"
    />

    <AInput
        v-if="showFilter"
        v-model:value="filters.age"
        placeholder="Age"
    />

    <AInput
        v-if="showFilter"
        v-model:value="filters.address"
        placeholder="Address"
    />

    <ASelect
        v-if="showFilter"
        v-model:value="filters.role"
        placeholder="Role"
        style="width: 120px"
    >
      <ASelectOption value="admin">Admin</ASelectOption>
      <ASelectOption value="moderator">Moderator</ASelectOption>
      <ASelectOption value="user">User</ASelectOption>
    </ASelect>
  </AFlex>
  <ATable
      size="small"
      :dataSource="filteredData"
      :columns="columns"
      :loading="loading"
      :error="error"
      :scroll="{ x: 'max-content' }"
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

<script lang ="ts">
import {
  Table as ATable,
  Tag as ATag,
  Button as AButton,
  InputSearch as AInputSearch,
  Flex as AFlex,
  Select as ASelect,
  Input as AInput,
} from "ant-design-vue";
import { onMounted, reactive, ref, computed } from "vue";

export default {
  components: { ATable, ATag, AButton, AInputSearch, AFlex, ASelect, AInput },
  setup() {
    const dataSource = ref([])
    const loading = ref(false)
    const error = ref(null)

    const showFilter = ref(false);

    const dataTable = ref([])

    const filters = reactive({
      search: '',
      field: '',
      address: '',
      firstName: '',
      lastName: '',
      age: '',
      role: '',
    })

    const fetchData = async () => {
      loading.value = true
      error.value = null
      try {
        const response = await fetch('https://dummyjson.com/users')

        if (!response.ok) {
          throw new Error('Ошибка запроса')
        }

        const data = await response.json()
        dataTable.value = data.users

      } catch (err) {
        error.value = err.message

      } finally {
        loading.value = false
      }
    }

    const filteredData = computed(() => {
      return dataTable.value.filter(user => {

        if (filters.search) {
          const value = filters.search.toLowerCase()

          const allMatch =
              user.firstName.toLowerCase().includes(value) ||
              user.lastName.toLowerCase().includes(value) ||
              String(user.age).includes(value) ||
              user.role?.toLowerCase().includes(value) ||
              user.address?.toLowerCase().includes(value)

          if (!allMatch) return false
        }

        if (filters.firstName && !user.firstName.toLowerCase().includes(filters.firstName.toLowerCase())) {
          return false
        }

        if (filters.lastName && !user.lastName.toLowerCase().includes(filters.lastName.toLowerCase())) {
          return false
        }

        if (filters.age && String(user.age) !== String(filters.age)) {
          return false
        }

        if (filters.address && !user.address.toLowerCase().includes(filters.address.toLowerCase())) {
          return false
        }

        return !(filters.role && user.role !== filters.role);
      })
    })

    onMounted(() => {
      fetchData()
    })

    const columns = [
      { title: 'ID', dataIndex: 'id', key: 'id', width: 80 },
      { title: 'First Name', dataIndex: 'firstName', key: 'firstName', width: 80 },
      { title: 'Last Name', dataIndex: 'lastName', key: 'lastName', width: 80 },
      { title: 'Maiden Name', dataIndex: 'maidenName', key: 'maidenName', width: 80 },
      { title: 'Age', dataIndex: 'age', key: 'age', width: 60 },
      { title: 'Gender', dataIndex: 'gender', key: 'gender', width: 80 },
      { title: 'Email', dataIndex: 'email', key: 'email', width: 80 },
      { title: 'Phone', dataIndex: 'phone', key: 'phone' },
      { title: 'Username', dataIndex: 'username', key: 'username', width: 60 },
      { title: 'Password', dataIndex: 'password', key: 'password', width: 80 },
      { title: 'Birth Date', dataIndex: 'birthDate', key: 'birthDate', width: 100 },
      { title: 'Image', dataIndex: 'image', key: 'image', width: 80 },
      { title: 'Blood Group', dataIndex: 'bloodGroup', key: 'bloodGroup', width: 80 },
      { title: 'Height', dataIndex: 'height', key: 'height', width: 80 },
      { title: 'Weight', dataIndex: 'weight', key: 'weight', width: 80 },
      { title: 'Eye Color', dataIndex: 'eyeColor', key: 'eyeColor', width: 80 },
      { title: 'Hair Color', dataIndex: ['hair', 'color'], key: 'hair.color', width: 80 },
      { title: 'Hair Type', dataIndex: ['hair', 'type'], key: 'hair.type', width: 80 },
      { title: 'IP', dataIndex: 'ip', key: 'ip', width: 80 },
      { title: 'Address', dataIndex: ['address', 'address'], key: 'address.address', width: 80 },
      { title: 'City', dataIndex: ['address', 'city'], key: 'address.city', width: 80 },
      { title: 'State', dataIndex: ['address', 'state'], key: 'address.state', width: 80 },
      { title: 'Postal Code', dataIndex: ['address', 'postalCode'], key: 'address.postalCode', width: 80 },
      { title: 'Country', dataIndex: ['address', 'country'], key: 'address.country', width: 80 },
      { title: 'University', dataIndex: 'university', key: 'university', width: 80 },
      { title: 'Company Name', dataIndex: ['company', 'name'], key: 'company.name', width: 80 },
      { title: 'Company Department', dataIndex: ['company', 'department'], key: 'company.department', width: 80 },
      { title: 'Company Title', dataIndex: ['company', 'title'], key: 'company.title', width: 80 },
      { title: 'Role', dataIndex: 'role', key: 'role', width: 80 },
    ]

    return {
      dataSource,
      loading,
      error,
      fetchData,
      columns,
      showFilter,
      filteredData,
      filters,
    };
  },
};
</script>

<style scoped lang="scss">
.table, .head {
  padding: 1rem;
}

.head {
  gap: 1rem;
}
</style>