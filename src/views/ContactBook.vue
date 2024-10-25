<script setup>
import { ref, computed, watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import ContactCard from '@/components/ContactCard.vue';
import InputSearch from '@/components/InputSearch.vue';
import ContactList from '@/components/ContactList.vue';
import MainPagination from '@/components/MainPagination.vue';
import contactsService from '@/services/contacts.service'; // Assuming this service is implemented

import { useMutation } from '@tanstack/vue-query';

// Router and Route instance
const router = useRouter();
const route = useRoute();

// Reactive states
const totalPages = ref(1);
const contacts = ref([]);
const selectedIndex = ref(-1);
const searchText = ref('');
const selectedContact = computed(() => (selectedIndex.value < 0 ? null : filteredContacts.value[selectedIndex.value]));

// Get the current page from the query string (?page=1)
const currentPage = computed(() => {
  const page = Number(route.query?.page);
  return !Number.isNaN(page) && page > 0 ? page : 1;
});

// Create a searchable string from each contact (name, email, address, phone)
const searchableContacts = computed(() => 
  contacts.value.map(contact => {
    const { name, email, address, phone } = contact;
    return `${name} ${email} ${address} ${phone}`;
  })
);

// Filter contacts based on the search text
const filteredContacts = computed(() => {
  if (!searchText.value) return contacts.value;
  return contacts.value.filter((contact, index) =>
    searchableContacts.value[index].toLowerCase().includes(searchText.value.toLowerCase())
  );
});

// Retrieve contacts for the specific page and order by name
const { mutate: fetchContacts, isLoading } = useMutation({
  mutationFn: (page) => contactsService.fetchContacts(page),
  onSuccess: (chunk) => {
    console.log(chunk)
    totalPages.value = chunk.metadata.lastPage ?? 1;
    contacts.value = chunk.contacts.sort((a, b) => a.name.localeCompare(b.name));
    selectedIndex.value = -1;
  },
  onError: (error) => {
    console.error('Error fetching contacts:', error);
  },
});


// Mutation to delete all contacts
const mutation = useMutation({
  mutationFn: async () => {
    await contactsService.deleteAllContacts();
  },
  onSuccess: () => {
    totalPages.value = 1;
    contacts.value = [];
    selectedIndex.value = -1;
    changeCurrentPage(1);
    console.log('All contacts deleted successfully.');
  },
  onError: (error) => {
    console.error('Error deleting all contacts:', error);
  },
});

async function refreshContacts() {
  await fetchContacts(currentPage.value);
}

// Handle deleting all contacts
async function onDeleteContacts() {
  if (confirm('Bạn muốn xóa tất cả Liên hệ?')) {
    mutation.mutate();
  }
}

// Navigate to the Add Contact page
function goToAddContact() {
  router.push({ name: 'contact.add' });
}

// Change the current page and update the query parameter
function changeCurrentPage(page) {
  router.push({ name: 'contactbook', query: { page } });
}

// Watch searchText and reset the selected index
watch(searchText, () => {
  selectedIndex.value = -1;
});

// Fetch contacts when currentPage changes
watch(currentPage, () => {
  fetchContacts(currentPage.value);
}, { immediate: true });
</script>

<template>
  <div class="page row mb-5">
    <div class="mt-3 col-md-6">
      <h4>
        Danh bạ
        <i class="fas fa-address-book"></i>
      </h4>

      <!-- Search Input -->
      <div class="my-3">
        <InputSearch v-model="searchText" />
      </div>

      <!-- Contact List -->
      <ContactList
        v-if="!isLoading && filteredContacts.length > 0"
        :contacts="filteredContacts"
        v-model:selected-index="selectedIndex"
      />
      <p v-else>
        Không có liên hệ nào.
      </p>

      <!-- Pagination and Actions -->
      <div class="mt-3 d-flex flex-wrap justify-content-round align-items-center">
        <MainPagination
          :total-pages="totalPages"
          :current-page="currentPage"
          @update:current-page="changeCurrentPage"
        />
        <div class="w-100"></div>
        <button class="btn btn-sm btn-primary" @click="refreshContacts">
          <i class="fas fa-redo"></i> Làm mới
        </button>
        <button class="btn btn-sm btn-success" @click="goToAddContact">
          <i class="fas fa-plus"></i> Thêm mới
        </button>
        <button class="btn btn-sm btn-danger" @click="onDeleteContacts">
          <i class="fas fa-trash"></i> Xóa tất cả
        </button>
      </div>
    </div>

    <div class="mt-3 col-md-6">
      <!-- Selected Contact Details -->
      <div v-if="selectedContact">
        <h4>
          Chi tiết Liên hệ
          <i class="fas fa-address-card"></i>
        </h4>
        <ContactCard :contact="selectedContact" />
        <router-link
          :to="{
            name: 'contact.edit',
            params: { id: selectedContact.id },
          }"
        >
          <span class="mt-2 badge text-bg-warning">
            <i class="fas fa-edit"></i> Hiệu chỉnh
          </span>
        </router-link>

      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  text-align: left;
  max-width: 750px;
}
</style>