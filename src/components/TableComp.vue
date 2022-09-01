<template>
    <div style="margin: 5px;">

        <template v-for="(s,k) in table.search">
            <template v-if="s.type=='slot'">
                <slot :name="s.slot_name"></slot>
            </template>
            <template v-else>
                <el-input :key="k" v-model="table.searchForm[s.prop]" :placeholder="s.placeholder" :style="{width:s.width+'px'}"
                          v-if="!s.static"></el-input>
            </template>

        </template>
        <div style="margin-left: 5px;display: inline-block;">
        <el-button type="primary" @click="load">搜索</el-button>
        <el-button type="primary" @click="reset">重置</el-button>
        </div>

        <div style="margin-left: 10px;display: inline-block;">
            <slot name="button_slot" :data="multipleSelection"></slot>
        </div>


        <el-table
                :data="table.data"
                @selection-change="handleSelectionChange"
                border
                style="width: 100%">


            <el-table-column
                    type="selection"
                    width="55"></el-table-column>

            <template v-for="(col,k) in table.columns">


                <el-table-column :prop="col.prop" :label="col.label" :width="col.width"  :key="k"
                                 v-if="col.type=='slot'">
                    <template slot-scope="scope">
                        <slot :name="col.slot_name" :row="scope.row"></slot>
                    </template>
                </el-table-column>
                <el-table-column :prop="col.prop" :label="col.label" :width="col.width"  :key="k"
                                 v-else-if="!col.hide">
                </el-table-column>
            </template>


          <!--  <el-table-column
                    prop="date"
                    label="日期"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="name"
                    label="姓名"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="address"
                    label="地址">
            </el-table-column>-->

        </el-table>
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="table.pageNum"
                :page-sizes="[5, 10, 20, 30]"
                :page-size="table.pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="table.total">
        </el-pagination>

        <template v-for="(f) in table.forms">
            <slot :name="f.slot_name"></slot>
        </template>
    </div>
</template>

<script>
    import axios from "axios";

    export default {
        name: "TableComp",
        data(){
            return {
                table:{
                    columns:[],
                    pageNum:1,
                    pageSize:10,
                    total:0,
                    http:{},
                    data:[],
                    search:[],
                    searchForm:{},
                    searchStaticForm:{},
                    forms:[]
                },
                multipleSelection:[]
            }
        },
        props:{
            data:Object
        },
        methods:{
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            buildParam(){
                let res={}
                res.pageSize=this.table.pageSize
                res.pageNum=this.table.pageNum

                for (const k in this.table.searchForm) {
                    res[k] = this.table.searchForm[k]
                }
                for (const k in this.table.searchStaticForm) {
                    res[k] = this.table.searchStaticForm[k]
                }


                return res
            },
            setSearchForm(o){
                for (let key in o) {
                    this.table.searchForm[key] = o[key]
                }
            },
            reset(){
                for (const k in this.table.searchForm) {
                    this.table.searchForm[k]=''
                }
            },
            load(){
               axios.post(this.table.http.url,this.buildParam()).then(res=>res.data).then(data=>{
                   console.log(data)
                   let result = this.table.http.result
                   //1. total 2.list
                   for (let key in result) {
                       if(key=='total'){
                           this.table.total=this.buildResult(data,result[key])  /* data['data.total']*/
                       }else if(key=='list'){
                           this.table.data=this.buildResult(data,result[key]) /* data['data.list']*/
                       }

                   }

               })
            },
            buildResult(data,key){
               let keys =  key.split('.');
                let res = data
                for (let k of keys) {
                    res = res[k]
                }

                return res
            },
            init(val){
                let form ={}
                let formStatic ={}
                for (let key in val) {
                    if(Object.keys(this.table).includes(key)){
                        this.table[key] = val[key]
                        if(key=='search'){

                            for (const k of val[key]) {

                                if(k.static){
                                    formStatic[k.prop] = k.default
                                }else{
                                    form[k.prop] = k.default
                                }
                            }
                            this.table.searchForm= form
                            this.table.searchStaticForm= formStatic
                        }

                    }
                }
            },
            handleSizeChange(val) {
                this.table.pageSize=val
                this.table.pageNum=1
                this.load()

                //console.log(`每页 ${val} 条`);
            },
            handleCurrentChange(val) {
                this.table.pageNum=val
                this.load()
               // console.log(`当前页: ${val}`);

            }
        },
        watch:{
            data:{
                handler(newVal){
                    this.init(newVal)
                },
                immediate:true,
                deep:true
            }
        },
        beforeMount() {
            this.load()
        }
    }
</script>

<style scoped>

</style>