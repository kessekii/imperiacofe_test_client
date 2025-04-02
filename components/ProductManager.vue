<template>
  <div class="product-manager">
    <h1>Product Management</h1>

    <!-- Filter Input -->
    <div class="filter-container">
      <input
        v-model="filter"
        placeholder="Search by name or category"
        class="filter-input"
      />
    </div>

    <!-- Product Table -->
    <table class="product-table">
      <thead>
        <tr>
          <th>Name</th>
          <th>Description</th>
          <th>Price</th>
          <th>Category</th>
          <th>Stock</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in paginatedProducts" :key="product._id">
          <td>{{ product.name }}</td>
          <td>{{ product.description }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.category }}</td>
          <td>{{ product.stock }}</td>
          <td>
            <button class="btn btn-edit" @click="editProduct(product)">Edit</button>
            <button class="btn btn-delete" @click="deleteProduct(product._id)">Delete</button>
          </td>
        </tr>
        <tr v-if="paginatedProducts.length === 0">
          <td colspan="6" class="no-results">No products found</td>
        </tr>
      </tbody>
    </table>
    <div class="pagination">
      <div class="pagination-info">
        Showing {{ startIndex + 1 }} to {{ Math.min(endIndex, filteredProducts.length) }} of {{ filteredProducts.length }} entries
      </div>
      
      <div class="pagination-controls">
        <button 
          @click="currentPage = 1" 
          class="pagination-btn" 
          :disabled="currentPage === 1"
          title="First page"
        >
          &laquo;
        </button>
        
        <button 
          @click="currentPage--" 
          class="pagination-btn" 
          :disabled="currentPage === 1"
          title="Previous page"
        >
          &lsaquo;
        </button>
        
        <template v-for="page in displayedPageNumbers" :key="page">
          <button 
            v-if="page !== '...'" 
            @click="currentPage = page" 
            :class="['pagination-btn', { active: currentPage === page }]"
          >
            {{ page }}
          </button>
          <span v-else class="ellipsis">{{ page }}</span>
        </template>
        
        <button 
          @click="currentPage++" 
          class="pagination-btn" 
          :disabled="currentPage >= totalPages"
          title="Next page"
        >
          &rsaquo;
        </button>
        
        <button 
          @click="currentPage = totalPages" 
          class="pagination-btn" 
          :disabled="currentPage >= totalPages"
          title="Last page"
        >
          &raquo;
        </button>
      </div>
      
      <div class="entries-selector">
        <label>
          Show 
          <select v-model="pageSize" @change="handlePageSizeChange">
            <option value="5">5</option>
            <option value="10">10</option>
            <option value="25">25</option>
            <option value="50">50</option>
            <option value="100">100</option>
          </select>
          entries
        </label>
      </div>
    </div>
    <!-- Product Form -->
    <div class="form-container">
      <h2>{{ isEditing ? 'Edit Product' : 'Add Product' }}</h2>
      <form @submit.prevent="saveProduct" class="product-form">
        <div class="form-group">
          <label for="name">Name:</label>
          <input id="name" v-model="form.name" required />
        </div>
        <div class="form-group">
          <label for="description">Description:</label>
          <input id="description" v-model="form.description" />
        </div>
        <div class="form-group">
          <label for="price">Price:</label>
          <input id="price" type="number" v-model="form.price" required />
        </div>
        <div class="form-group">
          <label for="category">Category:</label>
          <input id="category" v-model="form.category" />
        </div>
        <div class="form-group">
          <label for="stock">Stock:</label>
          <input id="stock" type="number" v-model="form.stock" />
        </div>
        <button type="submit" class="btn btn-submit">
          {{ isEditing ? 'Update' : 'Add' }}
        </button>
      </form>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

// Types
interface Product {
  _id: string;
  name: string;
  description?: string;
  price: number;
  category?: string;
  stock?: number;
}

interface FormState {
  _id: string | null;
  name: string;
  description?: string;
  price: number;
  category?: string;
  stock?: number;
}
const currentPage = ref<any>(1);
const pageSize = ref<number>(10);

// API Base URL
const API_BASE_URL = 'http://localhost:3400/';
const headers = {
        headers: {
          'Content-Type': 'application/json',
          "Authorization": `Bearer ${window?.localStorage.getItem('token')}`,
        },
      }
// State
const products = ref<Product[]>([]);
const filter = ref<string>('');
const form = ref<FormState>({
  _id: null,
  name: '',
  description: '',
  price: 0,
  category: '',
  stock: 0,
});
const isEditing = ref<boolean>(false);

// Computed: Filtered Products
const filteredProducts = computed<Product[]>(() =>
  products.value.filter(
    (product) =>
      product.name.toLowerCase().includes(filter.value.toLowerCase()) ||
      (product.category &&
        product.category.toLowerCase().includes(filter.value.toLowerCase()))
  )
);
// Computed: Total Pages
const totalPages = computed<number>(() => {
  return Math.max(1, Math.ceil(filteredProducts.value.length / pageSize.value));
});

// Computed: Start and End Index for current page
const startIndex = computed<number>(() => {
  return (currentPage.value - 1) * pageSize.value;
});

const endIndex = computed<number>(() => {
  return Math.min(startIndex.value + pageSize.value, filteredProducts.value.length);
});

// Computed: Paginated Products (current page only)
const paginatedProducts = computed<Product[]>(() => {
  return filteredProducts.value.slice(startIndex.value, endIndex.value);
});

// Computed: Page numbers to display
const displayedPageNumbers = computed(() => {
  const totalPagesNum = totalPages.value;
  const current = currentPage.value;
  const delta = 2; // Number of pages to show on each side of current page
  const result = [];
  
  // Always include first page
  result.push(1);
  
  // Calculate range around current page
  let rangeStart = Math.max(2, current - delta);
  let rangeEnd = Math.min(totalPagesNum - 1, current + delta);
  
  // Adjust range if current page is near the beginning
  if (current - delta <= 2) {
    rangeEnd = Math.min(totalPagesNum - 1, 1 + 2 * delta);
  }
  
  // Adjust range if current page is near the end
  if (current + delta >= totalPagesNum - 1) {
    rangeStart = Math.max(2, totalPagesNum - 2 * delta);
  }
  
  // Add ellipsis before range if needed
  if (rangeStart > 2) {
    result.push('...');
  }
  
  // Add range pages
  for (let i = rangeStart; i <= rangeEnd; i++) {
    result.push(i);
  }
  
  // Add ellipsis after range if needed
  if (rangeEnd < totalPagesNum - 1) {
    result.push('...');
  }
  
  // Always include last page if it's different from first page
  if (totalPagesNum > 1) {
    result.push(totalPagesNum);
  }
  
  return result;
});
// Fetch Products from API
async function fetchProducts(): Promise<void> {
  try {
    console.log('Fetching products...');
    const response = await axios.get<Product[]>(API_BASE_URL + 'products', 
      headers);
    products.value = response.data;
  } catch (error) {
    console.error('Error fetching products:', error);
  }
}
function handlePageSizeChange(): void {
  // Reset to first page when changing page size
  currentPage.value = 1;
}
// Save Product (Create or Update)
async function saveProduct(): Promise<void> {
  try {
    if (isEditing.value) {
      // Update Product
      await axios.put(`${API_BASE_URL}product?id=${form.value._id}`, form.value,
        headers
      );
      const index = products.value.findIndex((p) => p._id === form.value._id);
      if (index !== -1) {
        products.value[index] = { ...form.value } as Product;
      }
    } else {
      // Create Product
      const response = await axios.post<Product>(API_BASE_URL + 'product', form.value, 
        headers
      );
      products.value.push(response.data);
    }
    resetForm();
  } catch (error) {
    console.error('Error saving product:', error);
  }
}

// Edit Product
function editProduct(product: Product): void {
  form.value = { ...product };
  isEditing.value = true;
}

// Delete Product
async function deleteProduct(id: string): Promise<void> {
  try {
    await axios.delete(`${API_BASE_URL}product?id=${id}`, 
    headers);
    products.value = products.value.filter((product) => product._id !== id);
  } catch (error) {
    console.error('Error deleting product:', error);
  }
}

// Reset Form
function resetForm(): void {
  form.value = { _id: null, name: '', description: '', price: 0, category: '', stock: 0 };
  isEditing.value = false;
}

// Reset to first page when filter changes
watch(filter, () => {
  currentPage.value = 1;
});

// Fetch initial products on component mount
onMounted(fetchProducts);
</script>

<style lang="scss" scoped>
@use "~/components/ProductManager.scss";
</style>