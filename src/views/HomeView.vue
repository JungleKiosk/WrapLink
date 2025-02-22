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
            openedCategory: null
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
    <div class="container-fluid text-black">
        <div class="row">
            <div class="sidebar margin_top_sidbar">
                <!-- Sidebar -->
                <span class="btn btn-secondary m-3" @click="resetAllFilters">cancella filtri &#8634</span> <br>
                <p>I dati in tabella sono riferiti al file <br> Griglia_CU_O3_O4_V2.4 - v1.xls.</p>
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
                            <option v-for="inspire in uniqueInspire" :key="inspire" :value="inspire">{{ inspire }}
                            </option>
                        </select>
                        <div class="input-group-append">
                            <button class="btn btn-secondary" type="button"
                                @click="resetSelectedinspire">&#8634</button>
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

            <!-- Contenuto principale -->
            <div class="col-9 content-area">
                <div class="row">
                    <div class="col-12 table-container">
                        <table class="table table-striped">
                            <thead>
                                <tr>
                                    <th>ID</th>
                                    <th>Title</th>
                                    <th>Description</th>
                                    <th>URL</th>
                                    <th>CU</th>
                                    <th>inspire</th>
                                    <th>Stato</th>
                                    <th>Tags</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="link in filteredLinks" :key="link.id">
                                    <td>{{ link.id }}</td>
                                    <td>{{ link.title }}</td>
                                    <td>{{ link.description }}</td>
                                    <td> <a v-if="link.url" style="color: blue;" :href="link.url"
                                            target="_blank">WebSite</a>
                                        <span v-else>N/A</span>
                                    </td>
                                    <td><span class="badge bg-secondary">{{ link.cu }}</span></td>
                                    <td>
                                        <router-link to="/glossario" class=" text-light">
                                            <a class="text-primary" @click="openInspireModal(inspire)">
                                                {{ Array.isArray(link.inspire) ? link.inspire.join(', ') : link.inspire
                                                }}
                                            </a>
                                        </router-link>
                                    </td>
                                    <td>
                                        <span>Dati</span> <br>
                                        <span class="badge" :class="{
                                            'bg-success': link.stato === 'Noti e Accessibili',
                                            'bg-warning text-dark': link.stato === 'Noti ma non accessibili',
                                            'bg-danger': link.stato === 'Non Noti'
                                        }">
                                            {{ link.stato }}
                                        </span>
                                    </td>
                                    <td>
                                        <span v-for="tag in link.tags" :key="tag" class="text-white"
                                            :class="selectedTag === tag ? 'badge bg-primary' : 'badge bg-info text-dark mx-1'">
                                            {{ tag }}
                                        </span>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>



<style>
.margin_top_sidbar{
    margin-top: 7rem;
}

</style>