<script setup>
  
  import { notify } from "@kyvg/vue3-notification";
  // vue imports
  import { ref, createApp } from 'vue';
  import { storeToRefs } from 'pinia'

  // component imports
  import ListItem from '../../components/ListItem.vue'
  import CategoryModal from '../../components/CategoryModal.vue';
  import DeleteModal from '../../components/DeleteModal.vue';

  import LayerModal from '../../components/LayerModal.vue';
  import PublishModal from '../../components/PublishModal.vue';

  // store imports
  import { useCategoriesStore } from '../../stores/categories.store';
  import { useLayersStore } from '../../stores/layers.store';

  // logic
  const childComponentRef = ref(null);
  const deleteModalRef = ref(null);
  const publishModalRef = ref(null);

  const dynamicContainer = ref(null);
  let dynamicModal = null;
  let dynamicModalInstance = null;

  const lvl = ref(0);

  const categoriesStore = useCategoriesStore();
  const { categories } = storeToRefs(categoriesStore);
  categoriesStore.getAll();
  
  function addCategory(parent) {
    childComponentRef.value?.openModal(null, 'add', parent)
  }

  function openDeleteCategory(n) {
    deleteModalRef.value?.openModal(n,'category');
  }

  function deleteRecord(n, type) {
    if(type === 'category') {
      const categoriesStore = useCategoriesStore();
      const cat = n[0].external_id;
      categoriesStore.remove(cat)
        .then(res => {
          let arr = null;
          if(n && n.length > 1) {
            arr = n[1].categories;
          } else {
            arr = categories.value;
          }

          if(arr) {
            let index = arr.findIndex(i => i.external_id == n[0].external_id);
            if(index > -1) {
              arr.splice(index, 1);
            } 
          }

          notify({
            text: 'Categoría eliminada correctamente.',
            type: '!text-base !bg-green-600 !border-green-900',
          });

          deleteModalRef.value?.closeModal()
        }).catch(error => {
          console.log(error);
          notify({
            text: error?.response?.data?.message || 'Error al procesar la solicitud.',
            type: '!text-base !bg-red-500 !border-red-800',
          });
        });
    }

    if(type === 'layer') {
      const layersStore = useLayersStore();
      const lay = n.external_id;
      layersStore.remove(lay)
        .then(res => {
          let arr = n.layers;
          if(arr) {
            let index = arr.findIndex(i => i.external_id === n.external_id);
            if(index > -1) {
              arr.splice(index, 1);
            } 
          }

          notify({
            text: 'Capa eliminada correctamente.',
            type: '!text-base !bg-green-600 !border-green-900',
          });

          deleteModalRef.value?.closeModal()
        }).catch(error => {
          console.log(error);
          notify({
            text: error?.response?.data?.message || 'Error al procesar la solicitud.',
            type: '!text-base !bg-red-500 !border-red-800',
          });
        });
    }
  }

  function renderModal() {
    if(dynamicModal) {
      dynamicModal.unmount();
    }

    dynamicModal = createApp(LayerModal);
    dynamicModalInstance = dynamicModal.mount(dynamicContainer.value);
  }

  function renderPublishModal() {
    if(dynamicModal) {
      dynamicModal.unmount();
    }

    dynamicModal = createApp(PublishModal);
    dynamicModalInstance = dynamicModal.mount(dynamicContainer.value);
  }

  function editCategory(n) {
    childComponentRef.value?.openModal(n, 'edit', null)
  }

  function addLayer(parent) {
    renderModal()
    // layerModalRef.value?.openModal(null, 'add', parent)
    dynamicModalInstance?.openModal(null, 'add', parent);
  }

  function editLayer(n) {
    renderModal()
    // layerModalRef.value?.openModal(n, 'edit', null)
    dynamicModalInstance?.openModal(n, 'edit', null);
  }

  function openDeleteLayer(n) {
    deleteModalRef.value?.openModal(n,'layer');
  }

  function publish(item) {
    renderPublishModal()

    dynamicModalInstance?.openModal(item);
  }
</script>

<template>
  <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
    <div class="flow-root">
      <div class="-mx-4 -my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="inline-block min-w-full py-2 align-middle sm:px-6 lg:px-8">
          <div class="mb-6 flex items-center justify-start gap-x-5">
            <button type="button" @click="addCategory(null)"
              class="relative inline-flex items-center gap-x-1.5 rounded-md bg-blue-600 px-4 py-2 text-sm font-semibold text-white shadow-sm hover:bg-blue-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">
              <svg class="-ml-0.5 h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6.429 9.75L2.25 12l4.179 2.25m0-4.5l5.571 3 5.571-3m-11.142 0L2.25 7.5 12 2.25l9.75 5.25-4.179 2.25m0 0L21.75 12l-4.179 2.25m0 0l4.179 2.25L12 21.75 2.25 16.5l4.179-2.25m11.142 0l-5.571 3-5.571-3" />
              </svg>
              Agregar categoría
            </button>
            <RouterLink to="/" class="relative inline-flex items-center gap-x-1.5 rounded-md bg-green-600 px-4 py-2 text-sm font-semibold text-white shadow-sm hover:bg-green-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-green-600">
              Ir al mapa
            </RouterLink>
          </div>
          <div class="shadow ring-1 ring-black ring-opacity-5 sm:rounded-lg">
            <table class="min-w-full divide-y divide-gray-300">
              <thead class="bg-gray-50">
                <tr>
                  <th scope="col" class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6">Nombre</th>
                  <th scope="col" class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900"></th>
                  <th scope="col" class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Ult. Modif.</th>
                  <th scope="col" class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Estado</th>
                  <th scope="col" class="relative py-3.5 pl-3 pr-4 sm:pr-6">
                    <span class="sr-only">Acciones</span>
                  </th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200 bg-white">
                <tr v-if="categories?.loading">
                  <th colspan="5" class="bg-gray-100 py-2 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-3">
                    <svg class="animate-spin mx-auto h-8 w-8 text-blue-700" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                      <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                  </th>
                </tr>
                <ListItem 
                  v-if="categories.length" 
                  v-for="category in categories" 
                  :item="category" 
                  :lvl="lvl + 1" 
                  @delete="openDeleteCategory" 
                  @edit="editCategory" 
                  @add="addCategory"
                  @addLayer="addLayer"
                  @editLayer="editLayer"
                  @deleteLayer="openDeleteLayer"
                  @publish="publish"
                />
                <template v-if="!categories?.loading && categories.length == 0">
                  <tr>
                    <th colspan="5" class="p-5">
                      <div class="relative block w-full rounded-lg border-2 border-dashed border-gray-300 p-4 text-center">
                        <span class="block text-sm font-semibold italic text-gray-500">Añade una categoría para verla aquí.</span>
                      </div>
                    </th>
                  </tr>
                </template>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
    <CategoryModal 
      ref="childComponentRef" 
      @refresh="(n) => categories.push(n)" 
    />
    <DeleteModal 
      ref="deleteModalRef" 
      @confirmed="deleteRecord" 
    />
    <!-- <PublishModal 
      ref="publishModalRef"
    /> -->
    <!-- <LayerModal 
      ref="layerModalRef"
    /> -->
    <div ref="dynamicContainer"></div>
  </div>
</template>