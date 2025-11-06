<template>
    <div>
        <form @submit.prevent="onSubmit">
            <input v-model="name" placeholder="Enter product name" required />
            <input v-model="price" type="number" placeholder="Enter product price" required />
            <input v-model="category" placeholder="Enter product category" required />
            <button type="submit">{{ editingId ? 'Save' : 'Add Product' }}</button>
            <button v-if="editingId" type="button" @click="cancelEdit">Cancel</button>
        </form>

        <div v-if="products.length === 0">
            No products available.
        </div>

        <ul>
            <li v-for="product in products" :key="product._id">
                <strong>{{ product.name }}</strong> - ₦{{ product.price }} ({{ product.category }})
                <button @click="startEdit(product)">Edit</button>
                <button @click="deleteProduct(product._id)">Delete</button>
            </li>
        </ul>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const products = ref([])
const name = ref('')
const price = ref(0)
const category = ref('')

const editingId = ref(null)

const API_URL = import.meta.env.VITE_API_URL


const fetchProducts = async () => {
    try {
        const response = await fetch(`${API_URL}/api/products`)
        const data = await response.json()
        products.value = data
    } catch (error) {
        console.error('Error fetching products:', error)
    }
}

const fetchAProduct = async (id) => {
    try {
        const response = await fetch(`${API_URL}/api/products/${id}`)
        const data = await response.json()
        console.log('Fetched product:', data)
    } catch (error) {
        console.error('Error fetching product:', error)
    }
}

const addProduct = async () => {
    try {
        const response = await fetch(`${API_URL}/api/products`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                name: name.value,
                price: price.value,
                category: category.value
            })
        })
        const newProduct = await response.json()
        products.value.push(newProduct)
        name.value = ''
        price.value = 0
        category.value = ''
    } catch (error) {
        console.error('Error adding product:', error)
    }
}

const startEdit = (product) => {
    editingId.value = product._id
    name.value = product.name
    price.value = product.price
    category.value = product.category
}

const cancelEdit = () => {
    editingId.value = null
    name.value = ''
    price.value = 0
    category.value = ''
}

const onSubmit = async () => {
    if (editingId.value) {
        try {
            const response = await updateProduct(editingId.value, {
                name: name.value,
                price: price.value,
                category: category.value
            })
            cancelEdit()
        } catch (error) {
            console.error('Error updating product:', error)
        }
    } else {
        await addProduct()
    }
}

const updateProduct = async (id, updatedData) => {
    try {
        const response = await fetch(`${API_URL}/api/products/${id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(updatedData)
        })
        const updatedProduct = await response.json()
        const index = products.value.findIndex(product => product._id === id)
        if (index !== -1) {
            products.value[index] = updatedProduct
        }
    } catch (error) {
        console.error('Error updating product:', error)
    }
}

const deleteProduct = async (id) => {
    try {
        await fetch(`${API_URL}/api/products/${id}`, {
            method: 'DELETE'
        })
        products.value = products.value.filter(product => product._id !== id)
    } catch (error) {
        console.error('Error deleting product:', error)
    }
}

onMounted(() => {
    fetchProducts()
})
</script>

<style>
form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
button {
  cursor: pointer;
}
</style>