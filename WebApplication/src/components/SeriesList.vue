<script>
import axios from "axios"
import api from "../orthancApi"
import SeriesItem from "./SeriesItem.vue"
import { translateDicomTag } from "../locales/i18n"

export default {
    props: ['studyId', 'patientMainDicomTags', 'studyMainDicomTags', 'studySeries'],
    emits: ['deletedStudy'],
    data() {
        return {
            seriesInfo: {},
        };
    },
    computed: {
        sortedSeriesIds() {
            let keys = Object.keys(this.seriesInfo);
            keys.sort((a, b) => (parseInt(this.seriesInfo[a].MainDicomTags.SeriesNumber) > parseInt(this.seriesInfo[b].MainDicomTags.SeriesNumber) ? 1 : -1))
            return keys;
        }
    },
    watch: {
        studySeries(newValue, oldValue) {
            for (const series of this.studySeries) {
                this.seriesInfo[series["ID"]] = series;
            }
        }
    },
    methods: {
        columnTitle(tagName) {
            if (tagName == "instances_number") {
                return "# " + this.$i18n.t('instances');
            } else {
                return translateDicomTag(this.$i18n.t, this.$i18n.te, tagName);
            }
        },
        columnTooltip(tagName) {
            if (tagName == "instances_number") {
                return this.$i18n.t("instances_number");
            } else {
                return translateDicomTag(this.$i18n.t, this.$i18n.te, tagName);
            }
        },
        onDeletedSeries(seriesId) {
            delete this.seriesInfo[seriesId];
            if (Object.keys(this.seriesInfo).length == 0) {
                this.$emit("deletedStudy", this.studyId);
            } else {
                this.messageBus.emit('deleted-series-from-study-' + this.studyId);
            }
        }
    },
    components: { SeriesItem }
}
</script>

<template>
    <table class="table table-responsive table-sm series-table">
        <thead>
            <tr>
                <th width="2%" scope="col" class="series-table-header"></th>
                <th
                    width="7%"
                    scope="col"
                    class="series-table-header cut-text"
                    data-bs-toggle="tooltip"
                    :title="columnTooltip('SeriesNumber')"
                    >{{columnTitle('SeriesNumber')}}</th>
                <th
                width="30%"
                scope="col"
                class="series-table-header cut-text"
                data-bs-toggle="tooltip"
                :title="columnTooltip('SeriesDescription')"
                >{{columnTitle('SeriesDescription')}}</th>
                <th
                width="10%"
                scope="col"
                class="series-table-header cut-text"
                data-bs-toggle="tooltip"
                :title="columnTooltip('BodyPartExamined')"
                >{{columnTitle('BodyPartExamined')}}</th>
                <th
                width="11%"
                scope="col"
                class="series-table-header cut-text text-center"
                data-bs-toggle="tooltip"
                :title="columnTooltip('Modality')"
                >{{columnTitle('Modality')}}</th>
                <th
                width="5%"
                scope="col"
                class="series-table-header cut-text text-center"
                data-bs-toggle="tooltip"
                :title="columnTooltip('instances_number')"
                >{{columnTitle('instances_number')}}</th>
            </tr>
        </thead>
        <SeriesItem
            v-for="seriesId in sortedSeriesIds"
            :key="seriesId"
            :seriesId="seriesId"
            :seriesInfo="seriesInfo[seriesId]"
            :studyMainDicomTags="this.studyMainDicomTags"
            :patientMainDicomTags="this.patientMainDicomTags"
            @deletedSeries="onDeletedSeries"
        ></SeriesItem>
    </table>
</template>

<style>

.series-table>:not(:first-child) {
    border-top: 0px !important;
}

.series-table>:first-child {
    border-bottom: 2px !important;
    border-style: solid !important;
    border-color: black !important;
}

.series-table {
}

.series-table>:nth-child(odd) >* >* {
    background-color: var(--series-odd-bg-color);
}

.series-table>:nth-child(even) >* >* {
    background-color: var(--series-even-bg-color);
}

.series-table > tbody > tr:hover > *{
    background-color: var(--series-hover-color);
}

.series-table > tbody > tr.series-row-expanded:hover > * {
    background-color: var(--series-details-bg-color);
}
.series-table > tbody > tr.series-details-expanded:hover > * {
    background-color: var(--series-details-bg-color);
}


.series-table-header {
    text-align: left;
    padding-left: 10px;
}

.series-table td {
    text-align: left;
    padding-left: 10px;
}

</style>