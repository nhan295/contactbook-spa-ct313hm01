<script setup>
import { ref } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import ContactForm from '@/components/ContactForm.vue';
import contactsService from '@/services/contacts.service';

import { useQuery, useMutation } from '@tanstack/vue-query';

const props = defineProps({
  contactId: { type: String, required: true },
});

const router = useRouter();
const route = useRoute();
const message = ref('');

const { data: contact, error: fetchError } = useQuery({
  queryKey: ['contact', props.contactId],
  queryFn: () => contactsService.fetchContact(props.contactId),
  onError: (error) => {
    console.log(error);
    router.push({
      name: 'notfound',
      params: { pathMatch: route.path.split('/').slice(1) },
      query: route.query,
      hash: route.hash,
    });
  },
});

const { mutate: updateContact } = useMutation({
  mutationFn: async (contact) => {
    return await contactsService.updateContact(contact.get('id'), contact);
  },
  onSuccess: () => {
    message.value = 'Liên hệ được cập nhật thành công.';
  },
  onError: (error) => {
    console.error('Error updating contact:', error);
  },
});

const { mutate: deleteContact } = useMutation({
  mutationFn: async (id) => {
    return await contactsService.deleteContact(id);
  },
  onSuccess: () => {
    router.push({ name: 'contactbook' });
  },
  onError: (error) => {
    console.error('Error deleting contact:', error);
  },
});

function onUpdateContact(contact) {
  updateContact(contact);
}

function onDeleteContact(id) {
  if (confirm('Bạn muốn xóa Liên hệ này?')) {
    deleteContact(id);
  }
}
</script>

<template>
  <div v-if="contact" class="page">
    <h4>Hiệu chỉnh Liên hệ</h4>
    <ContactForm
      :contact="contact"
      @submit:contact="onUpdateContact"
      @delete:contact="onDeleteContact"
    />
    <p>{{ message }}</p>
  </div>
</template>