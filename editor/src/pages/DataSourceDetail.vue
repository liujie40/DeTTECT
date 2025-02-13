<template>
    <div class="card" v-if="dataSource != null">
        <auto-suggest-title
            title="Data source"
            :item="dataSource"
            itemIdName="data_source_name"
            :allItems="allDataSources"
            :suggestionList="makeDataSourceSuggestionList()"
            :navigateItem="navigateItem"
        ></auto-suggest-title>
        <applicable-to-collapse-data-sources
            :dataSource="dataSource"
            :allSystems="allSystems"
            helpText="..."
            :dqHelpText="dqHelpText"
            :dsHelpText="dsHelpText"
            :prevDataSourceQuality="prevDataSourceQuality"
            ref="collapseDataSourceComponent"
        ></applicable-to-collapse-data-sources>
    </div>
</template>

<script>
import ApplicableToCollapseDataSources from '@/components/Inputs/ApplicableToCollapseDataSources';
import AutoSuggestTitle from '@/components/Inputs/AutoSuggestTitle';
import dataSources from '@/data/data_sources';
import customDataSources from '@/data/dettect_data_sources';
import dataSourcePlatforms from '@/data/data_source_platforms';
import { pageDetailMixin } from '../mixins/PageDetailMixins.js';
import _ from 'lodash';

export default {
    data() {
        return {
            selectedPlatforms: Array
        };
    },
    computed: {
        getDataSources() {
            return dataSources[this.dataSourcePlatformsSelectorATTACK];
        },
        dataSourcePlatformsSelectorATTACK() {
            return this.domain == 'enterprise-attack' ? 'ATT&CK-Enterprise' : 'ATT&CK-ICS';
        },
        dataSourcePlatformsSelectorDETTECT() {
            return this.domain == 'enterprise-attack' ? 'DeTT&CT-Enterprise' : 'DeTT&CT-ICS';
        }
    },
    created: function () {
        this.getSelectedPlatforms();
    },
    mixins: [pageDetailMixin],
    props: {
        dataSource: {
            type: Object,
            required: true
        },
        allDataSources: {
            type: Array,
            required: true
        },
        dqHelpText: {
            type: String,
            required: true
        },
        dsHelpText: {
            type: String,
            required: true
        },
        prevDataSourceQuality: {
            type: Object,
            required: true
        },
        navigateItem: {
            type: Function,
            required: true
        },
        allSystems: {
            type: Array,
            required: true
        },
        domain: {
            type: String,
            required: true
        }
    },
    methods: {
        closeAllCollapses() {
            this.$refs.collapseDataSourceComponent.closeAllCollapses();
        },
        getSelectedPlatforms() {
            let selectedPlatforms = new Set();
            for (let i = 0; i < this.allSystems.length; i++) {
                for (let j = 0; j < this.allSystems[i].platform.length; j++) {
                    selectedPlatforms.add(this.allSystems[i].platform[j]);
                }
            }
            this.selectedPlatforms = Array.from(selectedPlatforms);
        },
        makeDataSourceSuggestionList() {
            // Make the data source suggestionlist based on both data sources and DeTT&CT data sources and check if the platform of these
            // (DeTT&CT) data sources corresponds to the selected platforms within the systems key-value pair.
            let suggestionList = new Set();
            for (let i = 0; i < this.selectedPlatforms.length; i++) {
                for (let j = 0; j < this.getDataSources.length; j++) {
                    if (
                        this.selectedPlatforms[i] == 'all' ||
                        dataSourcePlatforms[this.dataSourcePlatformsSelectorATTACK][this.selectedPlatforms[i]].includes(this.getDataSources[j])
                    ) {
                        suggestionList.add(this.getDataSources[j]);
                    }
                }

                for (let j = 0; j < customDataSources.length; j++) {
                    if (
                        this.selectedPlatforms[i] == 'all' ||
                        dataSourcePlatforms[this.dataSourcePlatformsSelectorDETTECT][this.selectedPlatforms[i]].includes(customDataSources[j])
                    ) {
                        suggestionList.add(customDataSources[j]);
                    }
                }
            }

            return Array.from(suggestionList).sort();
        }
    },
    components: {
        AutoSuggestTitle,
        ApplicableToCollapseDataSources
    }
};
</script>
