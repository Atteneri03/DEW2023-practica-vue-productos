<script setup>
import {computed} from 'vue';

const props = defineProps(['cardProducts']);

const total = computed(() => {
    return props.cardProducts.reduce((total,p) => total+= p.price * p.amount, 0);
});

const emit = defineEmits([
  'removeFromCard',
  'changeAmount',
  'decrementAmount',
  'incrementAmount'
]);

function onChangeAmount(product){
  if(product.amount <=0){
    emit('removeFromCard',product.id);
  } else {
    emit('changeAmount', product.id, product.amount);
  }
}

</script>

<template>
    <div>
    <h2>Carrito de compra</h2>
    <table>
      <thead>
        <tr>
          <th></th>
          <th>id</th>
          <th>Nombre</th>
          <th>Precio</th>
          <th>Cantidad</th>
          <th>Subtotal</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in cardProducts" :key="product.id">
            <td>
                <button @click="$emit('removeFromCard', product.id)">X</button>
            </td>
            <td>{{ product.id }}</td>
            <td>{{ product.name }}</td>
            <td>{{ product.price }}</td>
            <td>
            <input type="number" style="width: 60px;" v-model="product.amount" @change.lazy="onChangeAmount(product)">
            <button @click="$emit('incrementAmount',product.id)">+</button>
            <button @click="$emit('decrementAmount',product.id)">-</button>
            </td>
            <td>{{ (product.price * product.amount).toFixed(2) }}</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
            <th colspan="5">TOTAL</th>
            <td>{{ total.toFixed(2) }}</td>
        </tr>
      </tfoot>
</table>
</div>
</template>

<style scoped>
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
