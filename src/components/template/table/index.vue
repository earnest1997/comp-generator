<template>
    <div class="history">
        <el-form size="mini" :inline="true" ref="form" :model='form'>
            <!-- <el-form-item>
                <el-button type="primary" icon="el-icon-arrow-left" @click="back">返回</el-button>
            </el-form-item> -->


            <el-form-item>
                <el-button type="primary" @click="handleSearch">查询</el-button>
            </el-form-item>
            <el-form-item>
                <el-button @click="addNew">重置</el-button>
            </el-form-item>
        </el-form>

        <el-table border :data="tableData">
            <el-table-column align="center" v-for="(item,index) in columns" :key='index'></el-table-column>
        </el-table>

        <el-pagination class="pagination" background layout="total, prev, pager, next, sizes"
            @size-change="val => handlePageInfoChange('page_size', val)"
            @current-change="val => handlePageInfoChange('page', val)" :page-sizes="[20, 50, 100]" :page-size="20"
            :current-page.sync="page" :total="tableLen"></el-pagination>
    </div>
</template>

<script>
    import {
        getList
    } from './model.js'
    import dayjs from 'dayjs'

    import {
        getValueSafely
    } from '@utils'
        // config start
import {defaultConfig} from './config'
        const config=defaultConfig
    // config end

    const allItem = { label: '全部', value: '' }

    export default {
        data() {
            return {
                statusMap: {},
                tableData: [],
                tableLen: 0,
                page: 0,
                page_size: 20,
                typeMap: {},
                form: {},
                ...config
            }
        },
        mounted() {
            this.getDataAndRefresh()
        },
        methods: {
            addNew() {
               
            },

            edit(id) {
               
            },

            transformData(data) {
                return data.map(item => {
                    const {
                        status,
                        type,
                        ...rest
                    } = item
                    rest.status = (this.statusMap.find(item => item.value == status) || {}).label
                    return rest
                })
            },
            handleSearch() {
                this.page = 1
                this.getDataAndRefresh({
                    page_size: this.page_size
                })
            },
            handlePageInfoChange(key, val) {
                this[key] = val
                this.getDataAndRefresh({
                    page: this.page,
                    page_size: this.page_size
                })
            },
            getCurrentSearch() {
                const params = this.$refs.form.model
                const { time, ...rest } = params
                const [start, end] = time || []
                if (start) {
                    rest.start_time = dayjs(start).format('YYYY-MM-DD HH:mm:ss')
                    rest.end_time = dayjs(end).format('YYYY-MM-DD HH:mm:ss')
                }
                return rest
            },
            getDataAndRefresh(params = {
                page: 1
            }) {
                const param = {
                    ...params,
                    ...this.getCurrentSearch()
                }
                getList({
                    ...param
                }).then(res => {
                    const listData = getValueSafely(res, ['data', 'list']) || []
                    const listLen =
                        getValueSafely(res, ['data', 'page_info', 'total']) || 0
                    const statusMap = getValueSafely(res, ['data', 'status_map']) || {}

                    this.statusMap = [allItem, ...Object.entries(statusMap).map(([k, v]) => ({
                        label: v,
                        value: k
                    }))]
                    this.tableLen = +listLen
                    this.tableData = this.transformData(listData)
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .back {
        margin-bottom: 10px;
    }
</style>