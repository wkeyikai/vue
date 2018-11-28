<template>
    <div class="table-scope">
        <div class="relative">
            <div class="table-layout-fix" @scroll="scrollFunction">
                <div class="table-layout" ref="tableSlot">
                    <!--<list-loading ref="loading"></list-loading>-->
                    <table>
                        <thead>
                            <tr v-for="(row,idx) in ['tableTitle','tableTitleFixed']" ref="tableTitle" :key="idx">
                                <td v-for="(col,i) in title" :key="i">
                                    <sort-click v-if="col.sort" :title="col.label" v-on:sortby="sortBy" :keys="col.sort" :column="column" :type="type" ></sort-click>
                                    <template v-else>{{col.label}}</template>
                                </td>
                            </tr>
                        </thead>
                        <!--<tbody>
                            <tr v-for="(row,index) in data" :key="index">
                                <template v-for="col in grid">
                                    <slot :name="col" :vm="this" :row="row" :col="col" :text="row[col]">{{row[col]}}</slot>
                                </template>
                            </tr>
                            <tr v-show="data.length==0" class="noData"><td>{{$t('rp_no_data')}}</td></tr>
                        </tbody>-->
                        <template>
                        <table-view-tbody>
                        </table-view-tbody>
                        </template>
                        <!--<tbody>
                            <tr v-for="(row,index) in data" :key="index">
                                <template>
                                    <slot  :row="row" :grid="grid">
                                    </slot>
                                </template>
                            </tr>
                            <tr v-show="data.length==0" class="noData"><td>{{$t('rp_no_data')}}</td></tr>
                        </tbody>-->
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    components: {
        'table-view-tbody': {
            render (h) {
                // console.log('children',this.$parent.data)
                // console.log('>>',this.$parent.$slots.default)
                // console.log('this=>',this.$parent.$slots.default[0].data.attrs)
                // console.log('this=>',this.$parent.$slots.default[0].data.scopedSlots.default)
                // console.log('fun',fun)
                let slotsNames = []
                let slotMapping = {}
                let title = []
                let grids = this.$parent.grid
                let slotsDef = this.$parent.$slots.default.filter((v) => { return v.componentOptions && v.componentOptions.tag === 'table-view-col' })
                // console.log('slotsDef',slotsDef)
                slotsDef.forEach((slot) => {
                    slotMapping[slot.data.attrs.name] = slot
                })
                // console.log('map',slotMapping)
                grids.forEach((grid) => {
                    // console.log(grid,slotMapping[grid]);
                    let cr = slotMapping[grid]
                    if (cr) {
                        // title
                        let attr = slotMapping[grid].data.attrs
                        let label = attr.label || ''
                        let sort = attr.sort
                        title.push({name: grid, label: label, sort: sort})
                        // console.log(cr.data.staticClass)
                        // data slot
                        slotsNames.push({
                            name: grid,
                            // data:slotMapping[grid]?slotMapping[grid].data.scopedSlots:undefined,
                            cr: cr,
                            scopedSlots: cr.data.scopedSlots ? cr.data.scopedSlots.default : (name) => { return name.txt },
                            class: cr.data.staticClass ? {class: cr.data.staticClass} : {}
                        })
                    } else {
                        title.push({label: grid})
                        slotsNames.push({
                            name: grid,
                            // cr: cr,
                            scopedSlots: (name) => { return name.txt }
                            // class: cr.data.staticClass ? {class: cr.data.staticClass} : {}
                        })
                    }
                })
                // console.log('title',title);
                this.$parent.title = title
                let rowSlots = []
                this.$parent.data.forEach((val, i) => {
                    let slots = []
                    slotsNames.forEach((slot, j) => {
                        let props = {
                            index: i,
                            row: val,
                            txt: val[slot.name]
                        }
                        // console.log(slot.name,slot.cr.data.scopedSlots)
                        let txt = slot.scopedSlots(props)
                        slots.push(h('td', slot.class, txt))// h('td', vnode))
                    })
                    // @click="selected=entry" class="trigger" :class="{target:selected == entry}"
                    let trSet = {
                        /* on: {
                            click: function(e) {
                                console.log('click',e)
                            }
                        }, */
                        class: 'trigger'
                    }
                    rowSlots.push(h('tr', trSet, slots))
                })
                if (this.$parent.data.length === 0) {
                    rowSlots = [h('tr', {class: 'noData'}, [h('td', this.$t('rp_no_data'))])]
                }
                // slots = [h('tr', [h('td','1'),h('td','2')]),h('tr', [h('td','2'),h('td','1')])]
                return h('tbody', rowSlots)
            }
        },
        'sort-click': {
            props: ['title', 'column', 'keys', 'type'],
            template: `
                <div>
                    <a @click="sortBy(keys)" style="font-weight: bold;cursor: pointer; text-decoration: underline;color: #428BCA;">{{title}}</a>
                    <div class="sort">
                        <i class="fa fa-caret-up" v-if=" column!=keys || type == 'asc'"></i>
                        <i class="fa fa-caret-down" v-if=" column!=keys || type == 'desc'"></i>
                    </div>
                </div>
            `,
            methods: {
                sortBy (key) {
                    this.$emit('sortby', key)
                }
            }
        }
    },
    props: ['data', 'grid', 'column', 'type'],
    data () {
        return {
           title: []
        }
    },
    mounted () {
        this.$nextTick(() => {
            this.setFixedTitle()
        })
        window.onresize = () => {
            this.setFixedTitle()
        }
    },
    created () {
    },
    methods: {
        scrollFunction (e) {
            var titleFixed = this.$refs.tableTitle[1]
            titleFixed.style.left = 0 - e.srcElement.scrollLeft + 'px'
        },
        setFixedTitle () {
            if (!this.$refs.tableTitle) return
            var psTitle = this.$refs.tableTitle[0]
            var psFixTitle = this.$refs.tableTitle[1]
            var ps = psTitle.getElementsByTagName('td')
            var psFixed = psFixTitle.getElementsByTagName('td')
            psFixTitle.style.width = psTitle.clientWidth + 'px'
            for (let i = 0; i < ps.length; i++) {
                psFixed[i].style.width = ps[i].clientWidth + 'px'
            }
            if (this.$refs.tableFootTitle) {
                var psTitleFoot = this.$refs.tableFootTitle[0]
                var psFixTitleFoot = this.$refs.tableFootTitle[1]
                psFixTitleFoot.style.width = psTitleFoot.clientWidth + 'px'
                for (let i = 0; i < ps.length; i++) {
                    psFixed[i].style.width = ps[i].clientWidth + 'px'
                }
            }
        },
        setLoading (status) {
            this.$refs.loading.setShow(status)
        },
        getLoading () {
            return this.$refs.loading.getShow()
        },
        sortBy (key) {
            this.$emit('sortby', key)
        }
    },
    watch: {
        data () {
            this.$nextTick(() => {
                this.setFixedTitle()
            })
        }
    }
}
</script>
<style lang="scss">
.table-scope {
    height: 590px;
    width: 100%;
    overflow: hidden;
}
.relative {
    position: relative;
}
.table-layout-fix {
    width: 100%;
    height: 590px;
    overflow: auto;
}
.table-layout {
    min-width: unset;
}
.table-layout thead tr {
    width: 100%;
    background-color: #e5e5e5;
}
.table-layout thead tr:nth-of-type(2) {
    position: absolute;
    top: 0px;
    z-index: 100;
}
</style>
