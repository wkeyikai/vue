<template>
	<div>
		<table-view :data="gridData" :grid="['space','id','name','name1','name2','name3','name4','name5','name6']"  @sortby="sortBy" :column="sort.column" :type="sort.sort_type">
			<table-view-col name="space">
			</table-view-col>
			<table-view-col name="id" label="ID">
			</table-view-col>
			<table-view-col name="name" label="NAME"  sort="name_sort">
				<template slot-scope="{row}">
					{{row.id+':'+row.name}}
				</template>
			</table-view-col>
		</table-view>
	</div>
</template>
<script>
import tableView from '@/components/tableView'
import tableViewCol from '@/components/tableViewCol'
export default {
	name: 'Main',
	components: { tableView, tableViewCol },
	data () {
		return {
			demoRowData: {
				id: '1',
				name: 'Aaron',
				name1: 'aaron',
				name2: 'aaron',
				name3: 'aaron',
				name4: 'aaron',
				name5: 'aaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaronaaron',
				name6: 'aaron'
			},
			gridData: [],
			sort: {
				column: '',
				sort_type: ''
			}
		}
	},
	created () {
		for (let i = 0; i < 50; i++) {
			this.gridData.push(this.demoRowData)
		}
	},
	methods: {
		sortBy () {
		}
	}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
	font-weight: normal;
}
ul {
	list-style-type: none;
	padding: 0;
}
li {
	display: inline-block;
	margin: 0 10px;
}
a {
	color: #42b983;
}
</style>
