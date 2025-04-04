<script>
import SeriesDetails from "./SeriesDetails.vue";
import bootstrap from "bootstrap/dist/js/bootstrap.bundle.min.js"
import { mapState, mapGetters } from "vuex"
import SourceType from '../helpers/source-type';


export default {
    props: ["seriesId", "seriesInfo", 'studyMainDicomTags', 'patientMainDicomTags'],
    emits: ['deletedSeries'],
    data() {
        return {
            loaded: false,
            expanded: false,
            collapseElement: null
        };
    },
    computed: {
        ...mapState({
            studiesSourceType: state => state.studies.sourceType,
        }),
        instancesCount() {
            if (this.studiesSourceType == SourceType.LOCAL_ORTHANC) {
                return this.seriesInfo.Instances.length;
            } else if (this.studiesSourceType == SourceType.REMOTE_DICOM || this.studiesSourceType == SourceType.REMOTE_DICOM_WEB) {
                return this.seriesInfo.MainDicomTags.NumberOfSeriesRelatedInstances;
            }
        }
    },
    mounted() {
        this.$refs['series-collapsible-details'].addEventListener('show.bs.collapse', (e) => {
            if (e.target == e.currentTarget) {
                this.expanded = true;
            }
        });
        this.$refs['series-collapsible-details'].addEventListener('hide.bs.collapse', (e) => {
            if (e.target == e.currentTarget) {
                this.expanded = false;
            }
        });

        var el = this.$refs['series-collapsible-details'];
        this.collapseElement = new bootstrap.Collapse(el, {toggle: false});

        for (const [k, v] of Object.entries(this.$route.query)) {
            if (k === 'expand') {
                if (v === 'series' || v === 'instance') {
                    this.collapseElement.show();
                }
            }
        }
    },
    methods: {
        onDeletedSeries() {
            this.$emit("deletedSeries", this.seriesId);
        }
    },
    components: { SeriesDetails }
}
</script>


<template>
    <tbody>
        <tr :class="{ 'series-row-collapsed': !expanded, 'series-row-expanded': expanded }">
            <td></td>
            <td
                class="cut-text text-center"
                data-bs-toggle="collapse"
                v-bind:data-bs-target="'#series-details-' + this.seriesId"
            >{{ seriesInfo.MainDicomTags.SeriesNumber }}</td>
            <td
                class="cut-text"
                data-bs-toggle="collapse"
                v-bind:data-bs-target="'#series-details-' + this.seriesId"
            >
                <span
                    data-bs-toggle="tooltip"
                    v-bind:title="seriesInfo.MainDicomTags.SeriesDescription"
                >{{ seriesInfo.MainDicomTags.SeriesDescription }}</span>
            </td>
            <td
                class="cut-text text-center"
                data-bs-toggle="collapse"
                v-bind:data-bs-target="'#series-details-' + this.seriesId"
            >{{ seriesInfo.MainDicomTags.BodyPartExamined }}</td>
            <td
                class="cut-text text-center"
                data-bs-toggle="collapse"
                v-bind:data-bs-target="'#series-details-' + this.seriesId"
            >{{ seriesInfo.MainDicomTags.Modality }}</td>
            <td
                class="cut-text text-center"
                data-bs-toggle="collapse"
                v-bind:data-bs-target="'#series-details-' + this.seriesId"
            >{{ instancesCount }}</td>
        </tr>
        <tr
            class="collapse"
            :class="{ 'series-details-collapsed': !expanded, 'series-details-expanded': expanded }"
            v-bind:id="'series-details-' + this.seriesId"
            ref="series-collapsible-details"
        >
            <td colspan="100">
                <SeriesDetails
                    v-if="this.expanded"
                    :seriesId="this.seriesId"
                    :instancesIds="this.seriesInfo.Instances"
                    :seriesMainDicomTags="this.seriesInfo.MainDicomTags"
                    :studyMainDicomTags="this.studyMainDicomTags"
                    :patientMainDicomTags="this.patientMainDicomTags"
                    @deletedSeries="onDeletedSeries"
                ></SeriesDetails>
            </td>
        </tr>
    </tbody>
</template>

<style scoped>
.series-row-collapsed {
}

.series-row-expanded {
    background-color: var(--series-selected-color);
    font-weight: 700;

    border-top: 3px !important;
    border-style: solid !important;
    border-color: black !important;
}

.series-row-expanded>:first-child {
    border-bottom: 0px !important;
}

.series-details-expanded {
    background-color: var(--series-details-bg-color);

    border-top: 0px !important;
    border-bottom: 3px !important;
    border-style: solid !important;
    border-color: black !important;
}

</style>