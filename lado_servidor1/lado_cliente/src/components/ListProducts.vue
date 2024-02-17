<script setup>
import { ref, defineProps, computed, onMounted } from 'vue';

const searchWord = ref('');
const props = defineProps(['products']);

const filteredProducts = computed(() => {
  let products = props.products;
  if (categorySearch.value) {
    products = products.filter(p => p.category === categorySearch.value);
  }

  return products.filter(p =>
    p.name.toLowerCase().includes(searchWord.value.toLowerCase())
  ).sort((a, b) => {
    if (orderField.value === 'name') {
      return a.name.toLowerCase() < b.name.toLowerCase() ? -1 : 1;
    }
    if (orderField.value === 'price') {
      return a.price - b.price;
    }
    return 0;
  });
});

const orderField = ref('name');
const categories = ref([]);
const categorySearch = ref('');

onMounted(async () => {
  try {
    const response = await fetch('http://localhost:3000/api/categories');
    categories.value = (await response.json()).sort();
  } catch (error) {
    console.log('Error al cargar las categorías del servidor: ', error);
  }
});
</script>

<template>
  <div>
    <p>
      Filtrar por 
      <input type="text" v-model="searchWord">

      Ordenar por: 
      <input type="radio" id="name" value="name" v-model="orderField"/>
      <label for="name">Nombre</label>

      <input type="radio" id="price" value="price" v-model="orderField"/>
      <label for="price">Precio</label>
    </p>

    <p>Seleccionar categoría:
      <select v-model="categorySearch">
        <option value="">Cualquier categoría</option>
        <option v-for="category in categories" :key="category">{{ category }}</option>
      </select>
    </p>

    <div>
      <table v-if="filteredProducts.length">
        <thead>
          <tr>
            <th>id</th>
            <th>Nombre</th>
            <th>Categoría</th>
            <th>Precio</th>
            <th>Añadir a carrito</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in filteredProducts" :key="product.id" :class="{ warning: product.category === 'Alcoholic' }">
            <td>{{ product.id }}</td>
            <td>{{ product.name }}</td>
            <td>{{ product.category }}</td>
            <td>{{ product.price }}</td>
            <td><button @click="$emit('addToCard', product.id)">Añadir al Carrito</button></td>
            <td><button @click="$emit('deleteProduct',product.id)">Eliminar</button></td>
          </tr>
        </tbody>
      </table>
      <h3 v-else>No existen productos para esta categoría</h3>
    </div>
  </div>
</template>

<style>
table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px auto;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #FFB5A7;
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: #ffe1dc;
}

button {
 border: 2px solid #FFB5A7;
 background-color: #FFB5A7;
 border-radius: 0.9em;
 transition: all ease-in-out 0.1s;
 font-size: 14px;
}

button:active {
  color: #000000;
  box-shadow: inset 4px 4px 12px #da5050;
}


</style>