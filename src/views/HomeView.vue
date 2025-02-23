<script>
import linksData from '../data/links.json';
import { groupedTags } from '../data/groupTags.js';

export default {
    name: 'LinksView',
    data() {
        return {
            searchQuery: '',
            links: linksData,
            filteredLinks: linksData,
            groupedTags,
            selectedTag: '',
            selectedStato: '',
            selectedCu: '',
            selectedInspire: '',
            selectedInspireDetails: null,
            showModal: false,
            stati: ["Noti e Accessibili", "Noti ma non accessibili", "Non Noti"],
            debounceTimeout: null,
            tagSearchQuery: '',
            openedCategory: null,
            sidebarOpen: false
        };
    },
    created() {
        fetch('/src/data/links.json')
            .then(response => response.json())
            .then(data => {
                this.links = data;
                this.filteredLinks = data;
            })
            .catch(error => console.error('Errore nel caricamento dei dati:', error));
    },
    methods: {
        toggleSidebar() {
            this.sidebarOpen = !this.sidebarOpen;
        },
        debounceFilter() {
            if (this.debounceTimeout) {
                clearTimeout(this.debounceTimeout);
            }
            this.debounceTimeout = setTimeout(() => {
                this.filterLinks();
            }, 800);
        },
        onSearchInput() {
            this.selectedTag = '';
            this.debounceFilter();
        },
        filterLinks() {
            const query = this.searchQuery.toLowerCase();
            this.filteredLinks = this.links.filter(link => {
                const matchesQuery = link.title.toLowerCase().includes(query) ||
                    link.description.toLowerCase().includes(query) ||
                    link.tags.some(tag => tag.toLowerCase().includes(query));

                const matchesTag = !this.selectedTag || link.tags.includes(this.selectedTag);
                const matchesStato = !this.selectedStato || link.stato === this.selectedStato;

                const matchesCu = !this.selectedCu ||
                    (Array.isArray(link.cu) ? link.cu.includes(this.selectedCu) : link.cu === this.selectedCu);

                const matchesInspire = !this.selectedInspire ||
                    (Array.isArray(link.inspire) ? link.inspire.includes(this.selectedInspire) : link.inspire === this.selectedInspire);

                return matchesQuery && matchesTag && matchesStato && matchesCu && matchesInspire;
            });
        },
        filterByTag(tag) {
            this.searchQuery = '';
            this.selectedTag = (this.selectedTag === tag) ? '' : tag;
            this.filterLinks();
        },
        onStatoChange() {
            this.filterLinks();
        },
        toggleCategory(category) {
            this.openedCategory = this.openedCategory === category ? null : category;
        },
        resetSearchQuery() {
            this.searchQuery = '';
            this.filterLinks();
        },
        resetSelectedStato() {
            this.selectedStato = '';
            this.filterLinks();
        },
        resetSelectedCu() {
            this.selectedCu = '';
            this.filterLinks();
        },
        resetSelectedinspire() {
            this.selectedInspire = '';
            this.filterLinks();
        },
        resetAllFilters() {
            this.searchQuery = '';
            this.selectedStato = '';
            this.selectedCu = '';
            this.selectedInspire = '';
            this.filterLinks();
        },
        openInspireModal(inspire) {
            const foundInspire = inspireData.find(item => item.term === inspire);
            if (foundInspire) {
                this.selectedInspireDetails = foundInspire;
                this.showModal = true;
            }
        },
        closeInspireModal() {
            this.showModal = false;
            this.selectedInspireDetails = null;
        }
    },
    computed: {
        uniqueCu() {
            const cuValues = new Set();
            this.links.forEach(link => {
                if (Array.isArray(link.cu)) {
                    link.cu.forEach(cu => cuValues.add(cu));
                } else {
                    cuValues.add(link.cu);
                }
            });
            return Array.from(cuValues);
        },
        uniqueInspire() {
            const inspireValues = new Set();
            this.links.forEach(link => {
                if (Array.isArray(link.inspire)) {
                    link.inspire.forEach(inspire => inspireValues.add(inspire));
                } else {
                    inspireValues.add(link.inspire);
                }
            });
            return Array.from(inspireValues);
        }
    }
}
</script>

<template>
    <div class="container-fluid text-black mt-5">
        <button class="btn btn-primary toggle-btn" @click="toggleSidebar">☰ Filtri</button>

        <div class="sidebar margin_top_sidbar" :class="{ 'open': sidebarOpen }">
            <span class="btn btn-secondary m-3" @click="resetAllFilters">cancella filtri &#8634</span><br>
            <span>ATTENZIONE: i filtri sono incrociati!</span>
            <div class="col-6 m-3">
                <h5>Filtra Parola</h5>
                <div class="input-group">
                    <input class="search_bar form-control" type="text" v-model="searchQuery" placeholder="Cerca..."
                        @input="onSearchInput" />
                    <div class="input-group-append">
                        <button class="btn btn-secondary" type="button" @click="resetSearchQuery">&#8634</button>
                    </div>
                </div>
            </div>
            <div class="col-8 m-3">
                <h5>Filtra Stato</h5>
                <div class="input-group">
                    <select class="form-control" v-model="selectedStato" @change="onStatoChange">
                        <option value="">Tutti gli stati</option>
                        <option v-for="stato in stati" :key="stato" :value="stato">{{ stato }}</option>
                    </select>
                    <div class="input-group-append">
                        <button class="btn btn-secondary" type="button" @click="resetSelectedStato">&#8634</button>
                    </div>
                </div>
            </div>
            <div class="col-6 m-3">
                <h5>Filtra CU</h5>
                <div class="input-group">
                    <select class="form-control" v-model="selectedCu" @change="filterLinks">
                        <option value="">Tutti i CU</option>
                        <option v-for="cu in uniqueCu" :key="cu" :value="cu">{{ cu }}</option>
                    </select>
                    <div class="input-group-append">
                        <button class="btn btn-secondary" type="button" @click="resetSelectedCu">&#8634</button>
                    </div>
                </div>
            </div>
            <div class="col-6 m-3">
                <h5>INSPIRE</h5>
                <div class="input-group">
                    <select class="form-control" v-model="selectedInspire" @change="filterLinks">
                        <option value="">Attributi</option>
                        <option v-for="inspire in uniqueInspire" :key="inspire" :value="inspire">{{ inspire }}</option>
                    </select>
                    <div class="input-group-append">
                        <button class="btn btn-secondary" type="button" @click="resetSelectedinspire">&#8634</button>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col-8 m-3">
                    <h5>Filtra Categoria</h5>
                    <div v-for="group in groupedTags" :key="group.category" class="tag-group">
                        <button class="btn btn-secondary w-100 mb-2" @click="toggleCategory(group.category)">
                            {{ group.category }}
                        </button>
                        <div v-if="openedCategory === group.category" class="tag-grid">
                            <span v-for="tag in group.tags" :key="tag.id" class="badge tags"
                                :class="selectedTag === tag.name ? 'bg-primary' : 'bg-info'"
                                @click="filterByTag(tag.name)">
                                {{ tag.name }}
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Contenuto principale con CARD -->
        <div class="content-area">
            <div class="row">
                <div class="col-md-4" v-for="link in filteredLinks" :key="link.id">
                    <div class="card mb-4">
                        <div class="card-body">
                            <h5 class="card-title">{{ link.title }}</h5>
                            <p class="card-text">{{ link.description }}</p>
                            <a v-if="link.url" :href="link.url" target="_blank" class="btn btn-primary">Visita Sito</a>
                            <p v-else class="text-muted">N/A</p>
                            <p><strong>note:</strong> <span class="badge bg-secondary">{{ link.cu }}</span></p>
                            <div>
                                <strong>Tags:</strong>
                                <span v-for="tag in link.tags" :key="tag" class="badge bg-info text-dark mx-1">
                                    {{ tag }}
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
.margin_top_sidbar {
    margin-top: 7rem;
}

.sidebar {
    position: fixed;
    top: 0;
    left: -500px;
    width: 500px;
    height: 100%;
    background-color: #63b1ffea;
    transition: left 0.3s ease;
    z-index: 1000;
    padding: 2rem;
    overflow-y: auto;
}

.sidebar.open {
    left: 0;
}

.toggle-btn {
    position: fixed;
    top: 1rem;
    right: 1rem;
    z-index: 1100;
}

.card {
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    transition: transform 0.2s;
}

.card:hover {
    transform: translateY(-5px);
}

.card-title {
    font-size: 1.25rem;
    font-weight: bold;
}

.card-text {
    font-size: 1rem;
    color: #555;
}

.badge {
    font-size: 0.85rem;
}
</style>
