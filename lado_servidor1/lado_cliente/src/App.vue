<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import ListProducts from './components/ListProducts.vue';
import ShoppingCart from './components/ShoppingCart.vue';


const userName = ref('Atteneri');

const products = ref([]);


async function loadProducts() {
  try {
    const response = await fetch('http://localhost:3000/api/products');
    products.value = await response.json();
  } catch (error) {
    console.log('Error al cargar los productos del servidor: ',error);
  }
}

onMounted(() => {
  loadProducts();
  loadLocalCardIds();
});

const cardIds = ref([]);

const cardProducts = computed(() => {
  const cardProducts = [];

products.value.forEach(p => {
  const cardId = cardIds.value.filter(c => c.id==p.id)[0];
  if (cardId){
    cardProducts.push({
      id: p.id,
      name: p.name,
      price: p.price,
      amount: cardId.amount
    })
  }
});
  return cardProducts;
});

function addToCard(id){
  const cardId = cardIds.value.filter(c => c.id==id)[0];
  if(cardId){
    cardId.amount++;
  } else {
    cardIds.value.push({
      id:id,
      amount:1
    })
  }
}

function removeFromCard(id){
  cardIds.value = cardIds.value.filter(c => c.id!= id);
}


function decrementAmount(id){
  const cardId = cardIds.value.filter(c => c.id==id)[0];
  cardId.amount--;
  if(cardId.amount <= 0){
    removeFromCard(id);
  }
}

function incrementAmount(id){
  const cardId = cardIds.value.filter(c => c.id==id)[0];
  cardId.amount++;
}

function changeAmount(id, amount){
  const cardId = cardIds.value.filter(c => c.id==id)[0];
  cardId.amount = amount;
}

async function deleteProduct(id){
  console.log("Eliminado: " , id);
  try{
    const response = await fetch("http://localhost:3000/api/products/" + id, {method: 'DELETE'});
    if(response.ok && response.status==200){
      const product = await response.json();
      removeFromCard(product.id);
      products.value = products.value.filter(p => p.id != product.id);
    } else {
      throw("error al eliminar.");
    } 
  }catch(error){
    console.log("Error al eliminar del servidor: " , error);
  }
}

watch(cardIds, (newCardIds) => {
  localStorage.cardIds = JSON.stringify(newCardIds);
},{deep: true});


function loadLocalCardIds(){
  if(localStorage.cardIds){
    cardIds.value = JSON.parse(localStorage.cardIds);
  }
}

function removeShoppingCart(){
  cardIds.value = [];
}

async function sendOrder() {
  try{
    const requestOptions = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        user: userName.value,
        products: cardIds.value
      })
    }
    const response = await fetch("http://localhost:3000/api/orders" , requestOptions);
    if(response.ok && response.status==201){
      alert('Pedido realizado con éxito');
      removeShoppingCart();
    }else{
      throw('error al realizar la petición');
    } 
  }catch(error){
      console.log('Error al realizar la petición: ' , error);
    }
}

</script>

<template>
  <p>
  <h2>Practica Vue</h2>
  <input type="text" v-model="userName">
  <h2>Hola {{ userName }}</h2>
  </p>
  <div class="panel">
    <ListProducts 
    :products="products"
    @addToCard="addToCard"
    @deleteProduct="deleteProduct"
    />
    <ShoppingCart
    :cardProducts="cardProducts"
    @removeFromCard="removeFromCard"
    @incrementAmount="incrementAmount"
    @decrementAmount="decrementAmount"
    @changeAmount="changeAmount"
    />
  </div>
  
  <p>
    <button @click="sendOrder">Enviar Pedido</button>
  </p>
  
</template>



<style scoped>
.panel {
    display: flex;
    justify-content: space-between;
  }

</style>
