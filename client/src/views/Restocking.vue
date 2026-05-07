<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking Planner</h2>
      <p>Set a budget and get prioritized restock recommendations based on demand forecasts.</p>
    </div>

    <div v-if="loading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <!-- Budget Section -->
      <div class="card" style="margin-bottom: 24px;">
        <div class="card-header">
          <h3 class="card-title">Budget</h3>
        </div>
        <div class="budget-section">
          <div class="budget-slider-row">
            <label class="budget-label">Restock Budget</label>
            <span class="budget-value">{{ formatCurrency(budget) }}</span>
          </div>
          <input
            type="range"
            min="0"
            max="50000"
            step="500"
            v-model.number="budget"
            class="budget-slider"
          />
          <div class="budget-range-labels">
            <span>$0</span>
            <span>$50,000</span>
          </div>
          <div class="budget-summary" v-if="budget > 0">
            <span>{{ recommendedItems.length }} items recommended</span>
            <span class="summary-dot">·</span>
            <span>Estimated cost: <strong>{{ formatCurrency(selectedTotal) }}</strong></span>
            <span class="summary-dot">·</span>
            <span>Budget: <strong>{{ formatCurrency(budget) }}</strong></span>
          </div>
          <div class="budget-summary" v-else>
            <span>Set a budget to see recommendations.</span>
          </div>
        </div>
      </div>

      <!-- Success message -->
      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>

      <!-- Recommended Items Table -->
      <div v-if="budget > 0 && candidateItems.length > 0" class="card" style="margin-bottom: 24px;">
        <div class="card-header">
          <h3 class="card-title">Recommended Items ({{ recommendedItems.length }})</h3>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th style="width: 40px;"></th>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Trend</th>
                <th>Stock / Reorder</th>
                <th>Qty to Order</th>
                <th>Unit Cost</th>
                <th>Item Total</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in recommendedItems"
                :key="item.sku"
                :class="{ 'row-selected': selectedSkus.has(item.sku) }"
              >
                <td>
                  <input
                    type="checkbox"
                    :checked="selectedSkus.has(item.sku)"
                    @change="toggleSku(item.sku)"
                  />
                </td>
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ item.name }}</td>
                <td>
                  <span :class="['badge', getTrendBadgeClass(item.trend)]">{{ item.trend }}</span>
                </td>
                <td>{{ item.quantity_on_hand }} / {{ item.reorder_point }}</td>
                <td><strong>{{ item.restock_qty }}</strong></td>
                <td>{{ formatCurrency(item.unit_cost) }}</td>
                <td><strong>{{ formatCurrency(item.item_total_cost) }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Out of budget items -->
        <div v-if="outOfBudgetItems.length > 0">
          <div class="out-of-budget-heading">Outside budget ({{ outOfBudgetItems.length }} items)</div>
          <div class="table-container">
            <table>
              <tbody>
                <tr
                  v-for="item in outOfBudgetItems"
                  :key="item.sku"
                  class="row-out-of-budget"
                >
                  <td style="width: 40px;"></td>
                  <td><strong>{{ item.sku }}</strong></td>
                  <td>{{ item.name }}</td>
                  <td>
                    <span :class="['badge', getTrendBadgeClass(item.trend)]">{{ item.trend }}</span>
                  </td>
                  <td>{{ item.quantity_on_hand }} / {{ item.reorder_point }}</td>
                  <td><strong>{{ item.restock_qty }}</strong></td>
                  <td>{{ formatCurrency(item.unit_cost) }}</td>
                  <td><strong>{{ formatCurrency(item.item_total_cost) }}</strong></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Place Order button -->
        <div class="place-order-row">
          <button
            :disabled="budget === 0 || selectedItems.length === 0 || submitting"
            :style="placeOrderButtonStyle"
            @click="placeOrder"
          >
            <span v-if="submitting" class="spinner"></span>
            {{ submitting ? 'Placing Order...' : 'Place Order' }}
          </button>
          <span v-if="selectedItems.length > 0" class="order-summary-text">
            {{ selectedItems.length }} item(s) · Total: {{ formatCurrency(selectedTotal) }}
          </span>
        </div>
      </div>

      <!-- No candidates message -->
      <div v-else-if="budget > 0 && candidateItems.length === 0" class="card">
        <div class="empty-state">No items currently below reorder point. Inventory levels are healthy.</div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'

export default {
  name: 'Restocking',
  setup() {
    const loading = ref(true)
    const error = ref(null)
    const forecasts = ref([])
    const inventoryItems = ref([])
    const budget = ref(5000)
    const selectedSkus = ref(new Set())
    const submitting = ref(false)
    const successMessage = ref('')

    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Join inventory (left) with demand forecasts (right); default trend='stable' when no forecast
    const candidateItems = computed(() => {
      const forecastMap = new Map()
      for (const f of forecasts.value) {
        forecastMap.set(f.item_sku, f)
      }

      const candidates = []
      for (const inv of inventoryItems.value) {
        if (inv.quantity_on_hand >= inv.reorder_point) continue

        const forecast = forecastMap.get(inv.sku)
        const restock_qty = inv.reorder_point - inv.quantity_on_hand
        const item_total_cost = restock_qty * inv.unit_cost
        const depletionRatio = inv.reorder_point > 0 ? inv.quantity_on_hand / inv.reorder_point : 0

        candidates.push({
          sku: inv.sku,
          name: inv.name,
          category: inv.category,
          warehouse: inv.warehouse,
          quantity_on_hand: inv.quantity_on_hand,
          reorder_point: inv.reorder_point,
          unit_cost: inv.unit_cost,
          trend: forecast ? forecast.trend : 'stable',
          restock_qty,
          item_total_cost,
          depletionRatio
        })
      }

      const trendOrder = { increasing: 0, stable: 1, decreasing: 2 }
      candidates.sort((a, b) => {
        const trendDiff = (trendOrder[a.trend] ?? 3) - (trendOrder[b.trend] ?? 3)
        if (trendDiff !== 0) return trendDiff
        return a.depletionRatio - b.depletionRatio
      })

      return candidates
    })

    // Greedy inclusion up to budget
    const recommendedItems = computed(() => {
      if (budget.value <= 0) return []
      const result = []
      let cumulative = 0
      for (const item of candidateItems.value) {
        if (cumulative + item.item_total_cost <= budget.value) {
          result.push(item)
          cumulative += item.item_total_cost
        }
      }
      return result
    })

    const outOfBudgetItems = computed(() => {
      const recommendedSkus = new Set(recommendedItems.value.map(i => i.sku))
      return candidateItems.value.filter(i => !recommendedSkus.has(i.sku))
    })

    // Auto-select all recommended items when they change
    watch(recommendedItems, (items) => {
      selectedSkus.value = new Set(items.map(i => i.sku))
    })

    const selectedItems = computed(() =>
      recommendedItems.value.filter(i => selectedSkus.value.has(i.sku))
    )

    const selectedTotal = computed(() =>
      selectedItems.value.reduce((sum, i) => sum + i.item_total_cost, 0)
    )

    const toggleSku = (sku) => {
      const next = new Set(selectedSkus.value)
      if (next.has(sku)) {
        next.delete(sku)
      } else {
        next.add(sku)
      }
      selectedSkus.value = next
    }

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const filters = getCurrentFilters()
        const [forecastsData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({ warehouse: filters.warehouse, category: filters.category })
        ])
        forecasts.value = forecastsData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const placeOrder = async () => {
      if (selectedItems.value.length === 0 || budget.value === 0) return
      submitting.value = true
      successMessage.value = ''
      try {
        const filters = getCurrentFilters()
        const payload = {
          items: selectedItems.value.map(i => ({
            sku: i.sku,
            name: i.name,
            quantity: i.restock_qty,
            unit_price: i.unit_cost
          })),
          warehouse: filters.warehouse,
          total_value: selectedTotal.value
        }
        await api.submitRestockingOrder(payload)
        successMessage.value = `Restocking order placed successfully for ${selectedItems.value.length} item(s) totaling ${formatCurrency(selectedTotal.value)}.`
        selectedSkus.value = new Set()
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
        console.error(err)
      } finally {
        submitting.value = false
      }
    }

    const formatCurrency = (value) =>
      value.toLocaleString('en-US', { style: 'currency', currency: 'USD' })

    const getTrendBadgeClass = (trend) => {
      if (trend === 'increasing') return 'success'
      if (trend === 'stable') return 'info'
      if (trend === 'decreasing') return 'danger'
      return 'info'
    }

    const placeOrderButtonStyle = computed(() => {
      const isDisabled = budget.value === 0 || selectedItems.value.length === 0 || submitting.value
      return {
        background: '#2563eb',
        color: 'white',
        border: 'none',
        padding: '10px 20px',
        borderRadius: '6px',
        cursor: isDisabled ? 'not-allowed' : 'pointer',
        fontWeight: '500',
        fontSize: '14px',
        opacity: isDisabled ? '0.5' : '1',
        display: 'inline-flex',
        alignItems: 'center',
        gap: '8px'
      }
    })

    onMounted(loadData)
    watch([selectedLocation, selectedCategory], loadData)

    return {
      loading,
      error,
      budget,
      selectedSkus,
      submitting,
      successMessage,
      candidateItems,
      recommendedItems,
      outOfBudgetItems,
      selectedItems,
      selectedTotal,
      toggleSku,
      placeOrder,
      formatCurrency,
      getTrendBadgeClass,
      placeOrderButtonStyle
    }
  }
}
</script>

<style scoped>
.budget-section {
  padding: 1.5rem;
}

.budget-slider-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.budget-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #374151;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
}

.budget-slider {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  accent-color: #2563eb;
  cursor: pointer;
  margin-bottom: 0.5rem;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #94a3b8;
  margin-bottom: 1rem;
}

.budget-summary {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.875rem;
  color: #64748b;
  flex-wrap: wrap;
}

.budget-summary strong {
  color: #0f172a;
}

.summary-dot {
  color: #cbd5e1;
}

.success-message {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  margin-bottom: 24px;
  font-size: 0.875rem;
  font-weight: 500;
}

.out-of-budget-heading {
  padding: 0.75rem 1.5rem;
  font-size: 0.875rem;
  font-weight: 600;
  color: #94a3b8;
  border-top: 1px solid #f1f5f9;
  background: #f8fafc;
}

.row-out-of-budget td {
  opacity: 0.4;
}

.row-selected {
  background: #f0f9ff;
}

.place-order-row {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.25rem 1.5rem;
  border-top: 1px solid #e2e8f0;
}

.order-summary-text {
  font-size: 0.875rem;
  color: #64748b;
}

.spinner {
  display: inline-block;
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.4);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #64748b;
  font-size: 0.875rem;
}
</style>
