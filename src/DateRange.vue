<template>
	<div class="date-range">
		<div
			v-if="!noPresets"
			class="date-range__presets"
		>
			<v-list :dark="dark">
				<v-subheader>{{ labels.preset }}</v-subheader>
				<v-list-tile
					v-for="(preset, index) in presets"
					v-model="isPresetActive[index]"
					:key="index"
					@click="onPresetSelect(index)"
				>
					<v-list-tile-content>
						{{ preset.label }}
					</v-list-tile-content>
				</v-list-tile>
			</v-list>
		</div>
		<div class="date-range__pickers">
			<div class="date-range__picker date-range__pickers--start">
				<v-text-field
					v-model="formattedStartDate"
					:label="`${labels.start}(${format})`"
					name="startDate"
					class="date-range__pickers-input"
					:prepend-icon="prependIcon"
					readonly
				/>
				<v-date-picker
					:next-icon="nextIcon"
					:prev-icon="prevIcon"
					:dark="dark"
					:events="highlightRange ? dateRange.dates : events"
					:event-color="highlightRange ? dateRange.colors : eventColor"
					v-model="startDate"
					:min="options.minDate"
					:max="endDate"
					:locale="locale"
					:first-day-of-week="firstDayOfWeek"
					no-title
					@change="onDateRangeChange"
				/>
			</div>
			<div class="date-range__picker date-range__picker--end">
				<v-text-field
					:label="`${labels.end}(${format})`"
					v-model="formattedEndDate"
					name="endDate"
					class="date-range__pickers-input"
					readonly
				/>
				<v-date-picker
					:next-icon="nextIcon"
					:prev-icon="prevIcon"
					:dark="dark"
					:min="startDate"
					:max="maxDate"
					:events="highlightRange ? dateRange.dates : events"
					:event-color="highlightRange ? dateRange.colors : eventColor"
					v-model="endDate"
					:locale="locale"
					:first-day-of-week="firstDayOfWeek"
					no-title
					@change="onDateRangeChange"
				/>
			</div>
		</div>
	</div>
</template>

<script lang="ts">

/**
 * This component was based on the vuetify-daterange-picker from
 * https://github.com/praveenpuglia/vuetify-daterange-picker but it has
 * been updated to use the latest alpha version of date-fns (2.0.0-alpha.25),
 * and all the styles were converted to stylus language.
 * NOTE: for the custom styles to successfully override the default vuetify
 * styles you must use "stylus" as the language (in the style tag below).
 * For some reason, scss will not work, probably because webpack loads stylus
 * later and overwrites scss rules with stylus rules in the final css output.
 */

import { format, parse, addDays, subDays, addMonths, subMonths, differenceInDays, endOfDay } from 'date-fns'
const dateFmtString = 'yyyy-MM-dd'
const dateFmtDisplayString = 'MM/dd/yyyy'

export default {
	name: 'v-date-range',
	props: {
		options: {
			type: Object,
			required: true,
		},
		noPresets: {
			type: Boolean,
			default: false,
		},
		dark: {
			type: Boolean,
			default: false,
		},
		prependIcon: {
			type: String,
			default: 'event',
		},
		nextIcon: {
			type: String,
			default: 'chevron_right',
		},
		prevIcon: {
			type: String,
			default: 'chevron_left',
		},
		labels: {
			type: Object,
			default () {
				return {
					start: 'Start Date',
					end: 'End Date',
					preset: 'Presets',
				}
			},
		},
		events: {
			type: [Array, Object, Function],
			default: () => null,
		},
		eventColor: {
			type: [String, Function, Object],
			default: 'warning',
		},
		highlightRange: {
			type: Boolean,
			default: true,
		},
		highlightColors: {
			type: String,
			default: '',
		},
		locale: {
			type: String,
			default: 'en-us',
		},
		firstDayOfWeek: {
			type: Number,
			default: 0,
		},
	},
	data () {
		return {
			startDate: this.options.startDate,
			endDate: this.options.endDate,
			format: this.options.format,
			presets: this.options.presets,
			dateRange: {
				dates: [],
				colors: {},
			},
		}
	},
	computed: {
		formattedStartDate () {
			return this.startDate
				? format(parse(this.startDate, dateFmtString, new Date()), this.format)
				: ''
		},
		formattedEndDate () {
			return this.endDate ? format(parse(this.endDate, dateFmtString, new Date()), this.format) : ''
		},
		highlightColorClasses () {
			if (this.highlightColors) {
				return this.highlightColors
			}
			return this.dark ? 'blue-grey darken-1' : 'blue lighten-5'
		},
		isPresetActive () {
			return this.presets.map(
				preset =>
					preset.range[0] === this.startDate && preset.range[1] === this.endDate
			)
		},
		today () {
			return format(new Date(), dateFmtString)
		},
		maxDate () {
			return this.options.maxDate || this.today
		},
	},
	watch: {
		startDate () {
			this.onDateRangeChange()
		},
		endDate () {
			this.onDateRangeChange()
		},
	},
	mounted () {
		if (this.highlightRange) this.setInRangeData()
	},
	methods: {
		onPresetSelect (presetIndex) {
			this.startDate = this.presets[presetIndex].range[0]
			this.endDate = this.presets[presetIndex].range[1]
		},
		onDateRangeChange () {
			if (this.highlightRange) this.setInRangeData()
			this.$emit('input', [this.startDate, this.endDate])
		},
		setInRangeData () {
			const inRangeData = {
				dates: [],
				colors: {},
			}

			console.log('Hilighting date range!!!')
			if (this.highlightRange) {
				const startDate = parse(this.startDate, dateFmtString, new Date())
				const endDate = parse(this.endDate, dateFmtString, new Date())
				const diffDays = differenceInDays(endDate, startDate)

				for (let i = 0; i <= diffDays; i += 1) {
					const date = format(addDays(startDate, i), dateFmtString)
					inRangeData.dates.push(date)
					inRangeData.colors[date] = `date-range__date-in-range ${
						this.highlightColorClasses
					}`

					if (i === 0) inRangeData.colors[date] += ' date-range__range-start'
					if (i === diffDays) { inRangeData.colors[date] += ' date-range__range-end' }
				}
			}

			this.dateRange = inRangeData
		},

	},
}
</script>

<style scoped lang="stylus">

	.date-range {

		display: flex;

		&__presets {
			margin-right: 1rem;
		}

		&__pickers {
			display: flex;
		}

		&__picker {
			padding: 0 1rem;
		}

		>>> .date-picker-table,
		>>> .v-date-picker-table{
			table {
				border-collapse: collapse;
			}

			.btn,
			.v-btn {
				z-index: 1;
			}
			&__event {
				&.date-range {
					&__date-in-range {
						z-index: 0;
						/* override existing settings */
						width: 100%;
						height: 100%;
						left: 0;
						top: 0;
						bottom: 0;
						border-radius: 0;
						&.date-range {
							&__range-start {
								border-top-left-radius: 50%;
								border-bottom-left-radius: 50%;
								/* Cover only date button */
								left: 7px;
								width: 31px;
							}
							&__range-end {
								border-top-right-radius: 50%;
								border-bottom-right-radius: 50%;
							}
						}

					}
				}
			}

		}

	}

</style>