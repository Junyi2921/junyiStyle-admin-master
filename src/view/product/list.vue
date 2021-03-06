<template>
    <div class="tableDataList">
        <div class="tableDataListTools">
            <el-input class="otherStyle" v-model="searchName" placeholder="您可以按照名称/电话等信息搜索"></el-input>
            <el-select class="selectStyle" v-model="chooseProductType" clearable placeholder="商品类型">
                <el-option v-for="item in productType" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
            </el-select>
            <el-select class="selectStyle" v-model="chooseProductBrand" clearable placeholder="商品品牌">
                <el-option v-for="item in productBrand" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
            </el-select>
            <el-date-picker class="otherStyle" v-model="chooseTime" type="daterange" range-separator="至"
                            start-placeholder="开始日期" end-placeholder="结束日期">
            </el-date-picker>
            <el-button type="primary" icon="el-icon-search"></el-button>
            <el-button type="primary" icon="el-icon-edit" @click="goCreateView"></el-button>
            <el-button type="success" icon="el-icon-download" @click="setExcelAndDownload"></el-button>
        </div>
        <div class="tableDataListFilterBox">
            <div class="tableDataListFilterBoxIcon">
                <img src="/static/img/filter.svg">
            </div>
            <div class="tableDataListFilterBoxList">
                <CheckBoxItem checkBoxTitle="商品状态" :checkedList="productStateList"
                              v-model="chooseProductStateItems"></CheckBoxItem>
            </div>
        </div>

        <div class="tableDataListMain">
            <el-table :data="tableData" header-cell-class-name="headerClass" border style="width: 100%">
                <el-table-column fixed type="index" width="50" header-align="center" align="center"></el-table-column>
                <el-table-column fixed prop="productCode" label="商品编号" width="150" header-align="center"
                                 align="center"></el-table-column>
                <el-table-column prop="productName" label="商品名称" header-align="center" align="center" min-width="130">
                </el-table-column>
                <el-table-column prop="productBrand" label="商品品牌" width="120" header-align="center"
                                 align="center">
                    <template slot-scope="scope">
                        <el-tag type="info" close-transition>{{scope.row.productBrand}}</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="productModel" label="商品型号" width="120" header-align="center"
                                 align="center">
                    <template slot-scope="scope">
                        <el-tag type="warning" close-transition>{{scope.row.productBrand}}</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="productPrice" label="商品价格" width="120" header-align="center" align="center">
                    <template slot-scope="scope">
                        <el-tag type="danger" close-transition>
                            {{scope.row.productPrice}}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="productType" label="商品类型" width="120" header-align="center" align="center">
                    <template slot-scope="scope">
                        <el-tag :type="scope.row.productType === '采购' ? 'primary' : 'success'" close-transition>
                            {{scope.row.productType}}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="productState" label="商品状态" width="120" header-align="center" align="center">
                    <template slot-scope="scope">
                        <el-tag type="success" close-transition v-if="scope.row.productState === 'online'">
                            {{scope.row.productState | productStateCodeToName('scope.row.productState')}}
                        </el-tag>
                        <el-tag type="warning" close-transition v-if="scope.row.productState === 'offline'">
                            {{scope.row.productState | productStateCodeToName('scope.row.productState')}}
                        </el-tag>
                        <el-tag type="danger" close-transition v-if="scope.row.productState === 'delete'">
                            {{scope.row.productState | productStateCodeToName('scope.row.productState')}}
                        </el-tag>
                        <el-tag type="info" close-transition v-if="scope.row.productState === 'waitAuth'">
                            {{scope.row.productState | productStateCodeToName('scope.row.productState')}}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column fixed="right" label="操作" width="100" header-align="center" align="center">
                    <template slot-scope="scope">
                        <el-button @click="getItem(scope.row)" type="text" size="small">查看</el-button>
                        <el-button @click="editItem(scope.row)" type="text" size="small">编辑</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                        @size-change="handleSizeChange"
                        @current-change="handleCurrentChange"
                        :current-page="currentPage"
                        :page-sizes="[10, 20, 30, 40]"
                        :page-size="10"
                        layout="total, sizes, prev, pager, next, jumper"
                        :total="400">
                </el-pagination>
            </div>
        </div>

        <el-dialog title="商品详情" :visible.sync="productDetailReadVisible" width="80%">
            <ProductRead></ProductRead>
            <span slot="footer" class="dialog-footer">
                    <el-button @click="productDetailReadVisible = false">取 消</el-button>
                    <el-button type="primary" @click="productDetailReadVisible = false">确 定</el-button>
                 </span>
        </el-dialog>
    </div>
</template>

<style rel="stylesheet/scss" lang="scss">
    @import "../../commons/style/tableDataList.scss";
</style>

<script>
	import ProductRead from '@/view/product/read'
	import ExportToExcel from '@/tools/exportToExcel.js'
	import CheckBoxItem from '@/components/Other/CheckBox'

	export default {
		data() {
			return {
				productDetailReadVisible: false,
				currentPage: 1,
				chooseProductStateItems: [],
				productStateList: [{
					label: "已上架", value: "online"
				}, {
					label: "已下架", value: "offline"
				}, {
					label: "已删除", value: "delete"
				}, {
					label: "待审核", value: "waitAuth"
				}],
				searchName: "",
				chooseProductType: "",
				productType: [{
					label: "全部", value: "all"
				}, {
					label: "采购", value: "wholesale"
				}, {
					label: "零售", value: "retail"
				}],
				chooseProductBrand: "",
				productBrand: [{
					label: "全部", value: "all"
				}, {
					label: "商品品牌一", value: "brand01"
				}, {
					label: "商品品牌二", value: "brand02"
				}],
				chooseTime: "",
				tableData: []
			}
		}, methods: {
			changeChecked(val) {
				console.log(val);
			}, setExcelAndDownload() {
				let tHeader = ['商品编号', '商品名称', '商品品牌', '商品型号', '商品价格', '商品类型', '商品状态'];
				let filterVal = ['productCode', 'productName', 'productBrand', 'productModel', 'productPrice', 'productType', 'productState'];
				let dataList = this.tableData;
				ExportToExcel.exportJSON(tHeader, filterVal, dataList, '商品列表');
			}, handleSizeChange(val) {
				console.log(val);
			}, handleCurrentChange(val) {
				console.log(val);
			}, getItem(val) {
				console.log(val);
				this.productDetailReadVisible = true;
			}, goCreateView() {
				this.$router.push('/product/create');
			}, editItem(val) {
				this.$router.push('/product/edit');
			}
		}, components: {
			ProductRead, CheckBoxItem
		}, mounted() {
			this.tableData = [{
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}, {
				productCode: 'P201801290001',
				productModel: 'PM2921',
				productName: '商品名称01',
				productBrand: '商品品牌一',
				productType: '采购',
				productPrice: 2921.20,
				productState: "online"
			}];
		}, watch: {
			chooseProductStateItems(val) {
				console.log(val);
			}
		}
	}
</script>
