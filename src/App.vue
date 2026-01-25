<script setup>
import { ref, onMounted, computed, watch } from "vue";

// ===== STATE MANAGEMENT =====
const card = ref(null); // Default card shown on page load (Furret)
const searchQuery = ref(""); // User's search input
const searchResults = ref([]); // Array of search results from API
const selectedCard = ref(null); // Card that user clicks on from search results
const previewCard = ref(null); // Card being hovered over in search results
const isSearching = ref(false); // Loading state while searching
const searchError = ref(null); // Error message if search fails

// ===== UTILITY FUNCTIONS =====
// Debounce function - delays execution until user stops typing
// This prevents making an API call on every keystroke
const debounce = (fn, delay) => {
    let timeoutId;
    return (...args) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn(...args), delay);
    };
};

// ===== SEARCH FUNCTIONALITY =====
// Searches for cards by name using the TCGdex API
const search = async () => {
    searchError.value = null;

    // Only search if user typed 3+ characters
    if (searchQuery.value.length > 2) {
        isSearching.value = true;
        try {
            // Fetch search results from API
            const response = await fetch(
                `https://api.tcgdex.net/v2/en/cards?name=${searchQuery.value}`,
            );
            if (!response.ok) throw new Error("Search failed");

            // Parse JSON and store results
            const data = await response.json();
            searchResults.value = Array.isArray(data) ? data : [];
            isSearching.value = false;
        } catch (error) {
            console.error("Search error:", error);
            searchError.value = "Failed to search cards. Please try again.";
            searchResults.value = [];
            isSearching.value = false;
        }
    } else {
        // Clear results if search query is too short
        searchResults.value = [];
        isSearching.value = false;
    }
};

// Create debounced version of search (waits 300ms after user stops typing)
const debouncedSearch = debounce(search, 300);

// Watch for changes to search query and trigger search
watch(searchQuery, () => {
    if (searchQuery.value.length === 0) {
        // Immediately clear results if search is empty
        searchResults.value = [];
        isSearching.value = false;
    } else {
        // Otherwise, trigger debounced search
        debouncedSearch();
    }
});

// ===== CARD SELECTION =====
// When user clicks a search result, fetch full card details
const selectCard = async (result) => {
    // Clear search UI
    searchResults.value = [];
    searchQuery.value = "";
    previewCard.value = null;

    try {
        // Fetch complete card data (search results only have basic info)
        const response = await fetch(
            `https://api.tcgdex.net/v2/en/cards/${result.id}`,
        );
        if (!response.ok) throw new Error("Failed to load card details");

        const fullCardData = await response.json();
        selectedCard.value = fullCardData;
    } catch (error) {
        console.error("Error loading card details:", error);
        // Fallback to basic search result data if full fetch fails
        selectedCard.value = result;
    }
};

// ===== PREVIEW FUNCTIONALITY =====
// Show preview when hovering over a search result
const showPreview = (result) => {
    previewCard.value = result;
};

// Hide preview when mouse leaves search result
const hidePreview = () => {
    previewCard.value = null;
};

// Clear the selected card and return to default view
const clearSelection = () => {
    selectedCard.value = null;
};

// ===== COMPUTED PROPERTIES =====
// Generate the card image URL based on which card is active
const cardImageUrl = computed(() => {
    const activeCard = selectedCard.value || card.value;
    return activeCard ? `${activeCard.image}/high.webp` : "";
});

// ===== LIFECYCLE HOOKS =====
// Load default card (Furret) when component mounts
onMounted(async () => {
    try {
        const response = await fetch(
            "https://api.tcgdex.net/v2/en/cards/swsh3-136",
        );
        if (!response.ok) throw new Error("Failed to load default card");

        const data = await response.json();
        card.value = data;
    } catch (error) {
        console.error("Error loading default card:", error);
    }
});
</script>

<template>
    <div class="container">
        <!-- SEARCH SECTION -->
        <div class="search-section">
            <!-- Search input box -->
            <input
                type="text"
                v-model="searchQuery"
                placeholder="Search for a card... (min 3 characters)"
                class="search-input"
            />
            <!-- Show loading indicator while searching -->
            <span v-if="isSearching" class="loading-text">Searching...</span>
            <!-- Show error message if search fails -->
            <span v-if="searchError" class="error-text">{{ searchError }}</span>
        </div>

        <!-- SEARCH RESULTS (only shown when there are results) -->
        <div v-if="searchResults.length > 0" class="search-results">
            <!-- List of search results -->
            <ul class="results-list">
                <li
                    v-for="result in searchResults"
                    :key="result.id"
                    @click="selectCard(result)"
                    @mouseenter="showPreview(result)"
                    @mouseleave="hidePreview"
                    class="result-item"
                >
                    {{ result.name }}
                    <span class="result-id">({{ result.id }})</span>
                </li>
            </ul>
            <!-- Preview image shown when hovering over a result -->
            <div v-if="previewCard" class="preview-card">
                <img
                    :src="`${previewCard.image}/low.webp`"
                    :alt="previewCard.name"
                    class="preview-image"
                />
            </div>
        </div>

        <!-- CARD DISPLAY (shows either selected card or default card) -->
        <div v-if="selectedCard || card" class="card-display">
            <template v-if="selectedCard || card">
                <!-- Card name -->
                <h1>{{ (selectedCard || card).name }}</h1>

                <!-- Card image -->
                <img
                    :src="cardImageUrl"
                    :alt="(selectedCard || card).name"
                    class="card-image"
                />

                <!-- Card details (HP and weaknesses) -->
                <div class="card-info">
                    <!-- HP (if available) -->
                    <p v-if="(selectedCard || card).hp" class="hp">
                        HP: {{ (selectedCard || card).hp }}
                    </p>

                    <!-- Weaknesses (if available) -->
                    <div
                        v-if="(selectedCard || card).weaknesses?.length"
                        class="weaknesses"
                    >
                        <p
                            v-for="weakness in (selectedCard || card)
                                .weaknesses"
                            :key="weakness.type"
                            class="weakness"
                        >
                            Weak to {{ weakness.type }} {{ weakness.value }}
                        </p>
                    </div>
                </div>

                <!-- Button to clear selection (only shown when a card is selected) -->
                <button
                    v-if="selectedCard"
                    @click="clearSelection"
                    class="clear-button"
                >
                    Clear Selection
                </button>
            </template>
            <!-- Loading state -->
            <h1 v-else>Loading...</h1>
        </div>
    </div>
</template>

<style scoped>
.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.search-section {
    margin-bottom: 20px;
}

.search-input {
    width: 100%;
    padding: 12px;
    font-size: 16px;
    border: 2px solid rgb(69, 69, 71);
    border-radius: 8px;
    background-color: rgb(39, 39, 41);
    color: beige;
    box-sizing: border-box;
    transition: border-color 0.2s;
}

.search-input:focus {
    outline: none;
    border-color: rgb(100, 100, 102);
}

.loading-text {
    color: #4a9eff;
    font-size: 14px;
    margin-left: 10px;
}

.error-text {
    color: #ff6b6b;
    font-size: 14px;
    margin-left: 10px;
}

.search-results {
    display: flex;
    gap: 20px;
    margin-bottom: 20px;
}

.results-list {
    flex: 1;
    list-style-type: none;
    padding: 0;
    margin: 0;
    background-color: rgb(39, 39, 41);
    border-radius: 8px;
    max-height: 400px;
    overflow-y: auto;
}

.result-item {
    color: beige;
    padding: 12px;
    cursor: pointer;
    border-bottom: 1px solid rgb(69, 69, 71);
    transition: background-color 0.2s;
}

.result-item:last-child {
    border-bottom: none;
}

.result-item:hover {
    background-color: rgb(69, 69, 71);
}

.result-id {
    color: rgba(245, 245, 220, 0.6);
    font-size: 0.9em;
}

.preview-card {
    flex: 0 0 200px;
}

.preview-image {
    width: 100%;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.card-display {
    text-align: center;
}

h1 {
    color: rgb(255, 255, 255);
    margin-bottom: 20px;
    font-size: 2em;
}

.card-image {
    max-width: 400px;
    width: 100%;
    border-radius: 12px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
    margin: 20px auto;
    display: block;
}

.card-info {
    margin: 20px 0;
}

.hp {
    color: beige;
    font-size: 18px;
    font-weight: bold;
    margin: 10px 0;
}

.weaknesses {
    margin-top: 10px;
}

.weakness {
    color: beige;
    margin: 5px 0;
}

.clear-button {
    margin-top: 20px;
    padding: 10px 20px;
    background-color: rgb(69, 69, 71);
    color: beige;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.2s;
}

.clear-button:hover {
    background-color: rgb(89, 89, 91);
}
</style>

<style>
body {
    background-color: rgb(49, 49, 51);
    margin: 0;
    font-family: Arial, sans-serif;
}
</style>
