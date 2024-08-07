<template>
	<div class="tag">
		<div class="search gutter">
			<el-form :inline="true" :model="formInline" class="demo-form-inline">
				<el-form-item label="标签名称">
					<el-input
						v-model="formInline.tag_name"
						placeholder="请输入标签名称"
						clearable
					/>
				</el-form-item>
				<el-form-item label="标签类型">
					<el-select v-model="formInline.tag_type" class="w190" clearable>
						<el-option
							v-for="item in tagTypeData"
							:label="item.value"
							:value="item.key"
							:key="item.key"
						/>
					</el-select>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="onQuery">查询</el-button>
					<el-button @click="reset">重置</el-button>
				</el-form-item>
			</el-form>
		</div>
		<div class="table gutter">
			<el-table
				:data="tableData"
				ref="multipleTableRef"
				stripe
				style="width: 100%"
				@selection-change="handleSelectionChange"
				@row-click="showDetail"
			>
				<el-table-column type="selection" width="55" />
				<!-- <el-table-column prop="name"
          label="产品名称" /> -->
				<el-table-column prop="tagName" label="标签名称" />
				<el-table-column prop="tagType" label="标签类型" />
			</el-table>
		</div>
		<div class="action">
			<div class="left">
				<el-button @click="onClickAdd">新增</el-button>
				<el-button :disabled="!multipleSelection.length" @click="onHandleAction"
					>删除</el-button
				>
			</div>
			<div class="right">
				<el-pagination
					v-model:current-page="currentPage"
					:page-sizes="[10, 30, 50, 100]"
					:page-size="10"
					:background="true"
					layout="sizes, total, prev, pager, next"
					:total="totalPage"
					@current-change="handleCurrentChange"
					@size-change="handleSizeChange"
				/>
			</div>
		</div>
		<el-dialog v-model="dialogVisible" :title="title" width="400" center>
			<el-form :model="detailData" label-width="100">
				<el-form-item label="标签名称" required>
					<el-input
						v-model="detailData.tagName"
						placeholder="请输入标签名称"
						clearable
					/>
				</el-form-item>
				<el-form-item label="标签类型" required>
					<el-select v-model="detailData.tagType" class="w190" clearable>
						<el-option
							v-for="item in tagTypeData"
							:label="item.value"
							:value="item.key"
							:key="item.key"
						/>
					</el-select>
				</el-form-item>
			</el-form>
			<template #footer>
				<div class="dialog-action">
					<el-button type="primary" @click="onSaveData">确认</el-button>
					<el-button @click="dialogVisible = false">取消</el-button>
				</div>
			</template>
		</el-dialog>
	</div>
</template>

<script setup>
import { ref } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import { getTagList, getTagDetail, deleteTag, updateTag } from "@/network/api";
import { getTagTypeSelect } from "@/network/selectApi";

const formInline = ref({});
const currentPage = ref(1);
const totalPage = ref(0);
const multipleTableRef = ref();
const multipleSelection = ref([]);
const title = ref("新增标签");
const dialogVisible = ref(false);
const detailData = ref({});
const tagTypeData = ref([
	{
		value: "链接",
		key: "ITEM",
	},
	{
		value: "图片",
		key: "PHOTO",
	},
]);

const params = ref({
	size: 10,
});

const tableData = ref([]);

function onClickAdd() {
	title.value = "新增标签";
	detailData.value = {};
	dialogVisible.value = true;
}

async function showDetail(value) {
	title.value = "修改标签";
	detailData.value = await getTagDetail({ tag_id: value.id });
	dialogVisible.value = true;
}

async function onSaveData() {
	try {
		const res = await updateTag(detailData.value);
		ElMessage({
			message: res?.message ?? "操作成功",
			type: "success",
		});
		detailData.value = {};
		dialogVisible.value = false;
		getList();
	} catch (error) {}
}

function onQuery() {
	currentPage.value = 1;
	getList();
}

function reset() {
	formInline.value = {};
	onQuery();
}

function handleCurrentChange(val) {
	currentPage.value = val;
	getList();
}

function handleSizeChange(val) {
	params.value.size = val;
	currentPage.value = 1;
	getList();
}

async function getList() {
	const res = await getTagList({
		current: currentPage.value,
		...formInline.value,
		...params.value,
	});
	tableData.value = res?.recordList ?? [];
	totalPage.value = res?.count ?? 0;
}

function handleSelectionChange(val) {
	multipleSelection.value = val;
}

async function onHandleAction() {
	await ElMessageBox.confirm("确认删除?", "提示");
	const ids = multipleSelection.value.map((item) => item.id);
	const res = await deleteTag({ idList: ids });
	ElMessage({
		message: res?.message ?? "操作成功",
		type: "success",
	});
	getList();
}

async function init() {
	getList();
	tagTypeData.value = await getTagTypeSelect();
}
init();
</script>

<style lang="scss" scoped>
.tag {
	.w190 {
		width: 190px;
	}
	.gutter {
		margin-bottom: 20px;
	}
	.action {
		display: flex;
		justify-content: space-between;
	}
}
</style>
