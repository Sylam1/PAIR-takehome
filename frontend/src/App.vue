<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import {
  applyReviewAction,
  fetchReviewItems,
  type ReviewAction,
  type ReviewItem
} from "./api";

const currentReviewer = "alex";
const items = ref<ReviewItem[]>([]);
const selectedId = ref<string | null>(null);
const isLoading = ref(false);
const errorMessage = ref<string | null>(null);
const pendingAction = ref<ReviewAction | null>(null);

// filter types (only three are possible)
type RiskLevel = "high" | "medium" | "low";

// state of selected risk, defaulting to high (first shown to user)
const selectedRisk = ref<RiskLevel>("high");

// filter items based on the active tab, lower now used as fall back
const filteredItems = computed(() => {
  return items.value.filter(
    (item) => item.risk_level.toLowerCase() === selectedRisk.value
  );
});

const selectedItem = computed(() =>
  items.value.find((item) => item.id === selectedId.value) ?? items.value[0] ?? null
);

// function to switch tabs and reset the active item focus as we switch
function setRiskFilter(risk: RiskLevel) {
  selectedRisk.value = risk;
  const firstInFiltered = items.value.find(
    (item) => item.risk_level.toLowerCase() === risk
  );
  selectedId.value = firstInFiltered?.id ?? null;
}

// function for assigning css 
function getRiskClass(risk: string) {
  const norm = risk.toLowerCase();
  if (norm === "high") return "risk-high";
  if (norm === "medium") return "risk-medium";
  if (norm === "low") return "risk-low";
  return "";
}

// function for calculating number of tickets in certain risk category
function getTicketCount(risk: RiskLevel): number {
  return items.value.filter(
    (item) => item.risk_level.toLowerCase() === risk
  ).length;
}

async function loadItems() {
  isLoading.value = true;
  errorMessage.value = null;

  try {
    items.value = await fetchReviewItems();
    selectedId.value = selectedItem.value?.id ?? null;
  } catch (error) {
    errorMessage.value = "Something went wrong loading the queue.";
  } finally {
    isLoading.value = false;
  }
}

async function performAction(action: ReviewAction) {
  if (!selectedItem.value) return;

  pendingAction.value = action;
  errorMessage.value = null;

  try {
    const updated = await applyReviewAction(selectedItem.value.id, action, currentReviewer);
    items.value = items.value.map((item) => (item.id === updated.id ? updated : item));
  } catch (error) {
    errorMessage.value = "That action could not be completed.";
  } finally {
    pendingAction.value = null;
  }
}

function formatDate(value: string) {
  return new Intl.DateTimeFormat("en-GB", {
    dateStyle: "medium",
    timeStyle: "short"
  }).format(new Date(value));
}

onMounted(loadItems);
</script>

<template>
  <main class="page-shell">
    <header class="topbar">
      <div>
        <p class="eyebrow">Reviewer workspace</p>
        <h1>Active queue</h1>
      </div>
      <div class="reviewer">Signed in as {{ currentReviewer }}</div>
    </header>

    <p v-if="errorMessage" class="error-banner">{{ errorMessage }}</p>
    <p v-if="isLoading" class="loading">Loading review items...</p>

    <section v-else class="workspace">
      <aside class="queue-list" aria-label="Review queue">
        
        <nav class="risk-tabs" aria-label="Filter by risk">
          <button
            v-for="risk in (['high', 'medium', 'low'] as const)"
            :key="risk"
            type="button"
            class="tab-button"
            :class="{ active: selectedRisk === risk }"
            @click="setRiskFilter(risk)"
          >
            {{ risk.charAt(0).toUpperCase() + risk.slice(1) }}
            <span class="tab-count">({{ getTicketCount(risk) }})</span>
          </button>
        </nav>

        <div class="queue-items-wrapper">
          <button
            v-for="item in filteredItems"
            :key="item.id"
            class="queue-item"
            :class="{ selected: item.id === selectedItem?.id }"
            type="button"
            @click="selectedId = item.id"
          >
            <span class="queue-title">{{ item.title }}</span>
            <span class="queue-meta">
              <strong :class="getRiskClass(item.risk_level)">{{ item.risk_level }} risk</strong> 
              · {{ item.customer_tier }}
            </span>
            <span class="queue-meta">{{ item.status }} · {{ item.assigned_reviewer ?? "unassigned" }}</span>
          </button>
        </div>
      </aside>

      <section v-if="selectedItem" class="detail-panel">
        <div class="detail-header">
          <div>
            <p class="eyebrow">{{ selectedItem.id }}</p>
            <h2>{{ selectedItem.title }}</h2>
          </div>
          <span class="status-pill">{{ selectedItem.status }}</span>
        </div>

        <dl class="facts">
          <div>
            <dt>Submitted</dt>
            <dd>{{ formatDate(selectedItem.submitted_at) }}</dd>
          </div>
          <div>
            <dt>Risk</dt>
            <dd>{{ selectedItem.risk_level }}</dd>
          </div>
          <div>
            <dt>Customer</dt>
            <dd>{{ selectedItem.customer_tier }}</dd>
          </div>
          <div>
            <dt>Assignee</dt>
            <dd>{{ selectedItem.assigned_reviewer ?? "None" }}</dd>
          </div>
        </dl>

        <p class="summary">{{ selectedItem.summary }}</p>
        <p class="notes">{{ selectedItem.notes_count }} notes on this item</p>

        <div class="actions" aria-label="Workflow actions">
          <button type="button" :disabled="Boolean(pendingAction)" @click="performAction('claim')">
            Claim
          </button>
          <button type="button" :disabled="Boolean(pendingAction)" @click="performAction('approve')">
            Approve
          </button>
          <button type="button" :disabled="Boolean(pendingAction)" @click="performAction('reject')">
            Reject
          </button>
          <button type="button" :disabled="Boolean(pendingAction)" @click="performAction('escalate')">
            Escalate
          </button>
        </div>
      </section>
    </section>
  </main>
</template>
