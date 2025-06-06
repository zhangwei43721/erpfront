<template>
  <h2>商品信息</h2>
  <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
    <el-button type="primary" @click="openItemDialog">添加商品</el-button>
    <el-form :inline="true" :model="searchForm" class="search-form" ref="searchFormRef">
      <el-form-item label="商品编号" prop="itemNum">
        <el-input v-model="searchForm.itemNum" placeholder="请输入商品编号" clearable />
      </el-form-item>
      <el-form-item label="商品名称" prop="itemName">
        <el-input v-model="searchForm.itemName" placeholder="请输入商品名称" clearable />
      </el-form-item>
      <el-form-item label="状态" prop="statue">
        <el-select v-model="searchForm.statue" placeholder="请选择" clearable style="width: 100px;">
          <el-option label="上架" :value="0" />
          <el-option label="下架" :value="1" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleSearch">查询</el-button>
        <el-button @click="resetSearchForm">重置</el-button>
      </el-form-item>
    </el-form>
  </div>

  <!-- 商品列表表格 -->
  <el-table :data="itemList" row-key="id" stripe style="width: 100%">
    <el-table-column type="expand">
      <template #default="{ row }">
        <el-descriptions :column="2" border style="margin: 10px 20px;">
          <el-descriptions-item label="商品图片">
            <template v-if="row.imgs && Array.isArray(row.imgs) && row.imgs.length > 0">
              <div class="image-preview">
                <el-image v-for="(img, index) in row.imgs" :key="index" :src="img" :preview-src-list="row.imgs"
                  :initial-index="index" :preview-teleported="true" :z-index="3000" class="table-image" fit="cover"
                  style="width: 60px; height: 60px; margin-right: 5px; border-radius: 4px;"
                  @error="() => handleImageError(index)">
                  <template #error>
                    <div
                      style="width: 60px; height: 60px; display: flex; align-items: center; justify-content: center; background: #f5f7fa; color: #909399; font-size: 12px;">
                      加载失败<br />或无图片
                    </div>
                  </template>
                </el-image>
              </div>
            </template>
            <span v-else>无图片</span>
          </el-descriptions-item>
          <el-descriptions-item label="商品描述">{{ row.itemDesc }}</el-descriptions-item>
          <el-descriptions-item label="进货价格">{{ row.price }}</el-descriptions-item>
          <el-descriptions-item label="会员价格">{{ row.vipPrice }}</el-descriptions-item>
          <el-descriptions-item label="供应商">{{ row.supplyName }}</el-descriptions-item>
          <el-descriptions-item label="产地">{{ row.placeName }}</el-descriptions-item>
          <el-descriptions-item label="单位">{{ row.unitName }}</el-descriptions-item>
          <el-descriptions-item label="所属仓库">{{ row.storeName }}</el-descriptions-item>
          <el-descriptions-item label="生产日期">{{ row.itemDate }}</el-descriptions-item>
          <el-descriptions-item label="到期日期">{{ row.endDate }}</el-descriptions-item>
          <el-descriptions-item label="促销标题">{{ row.hotTitle }}</el-descriptions-item>
          <el-descriptions-item label="制造商">{{ row.facturer }}</el-descriptions-item>
          <el-descriptions-item label="创建者">{{ row.createBy }}</el-descriptions-item>
        </el-descriptions>
      </template>
    </el-table-column>
    <el-table-column label="商品编号" prop="itemNum" />
    <el-table-column label="商品名称" prop="itemName" />
    <el-table-column label="商品类型" prop="cateName" />
    <el-table-column label="品牌" prop="brandName" />
    <el-table-column label="库存" prop="store" />
    <el-table-column label="销售价格" prop="sellPrice" />
    <el-table-column label="状态" prop="statue">
      <template #default="{ row }">
        <el-tag :type="row.statue === 0 ? 'success' : 'danger'">
          {{ row.statue === 0 ? '上架' : '下架' }}
        </el-tag>
      </template>
    </el-table-column>

    <el-table-column fixed="right" label="操作" width="240">
      <template #default="{ row }">
        <el-button link size="small" type="primary" @click="handleDeleteItem(row.id)">删除</el-button>
        <el-button link size="small" type="primary" @click="openUpdateDialog(row)">修改</el-button>
        <el-button link size="small" type="primary" @click="openPurchaseDialog(row)">采购</el-button>
        <el-button link type="primary" size="small" @click="openOutDialog(row)">出库</el-button>
        <el-button v-if="row.statue === 1" link size="small" type="success" @click="handleUpItem(row.id)">上架</el-button>
        <el-button v-if="row.statue === 0" link size="small" type="warning"
          @click="handleDownItem(row.id)">下架</el-button>
      </template>
    </el-table-column>
  </el-table>

  <!-- 分页器 -->
  <el-pagination :page-size="10" :pager-count="5" :total="total" background class="mt-4" layout="prev, pager, next"
    size="small" @current-change="handlePageChange" />

  <!-- 添加商品对话框 -->
  <el-dialog v-model="dialogItemVisible" :width="dialogWidth" title="添加商品">
    <div style="margin-bottom: 20px;">
      <span>商品图片:</span>
      <el-upload :action="uploadImageUrl" :auto-upload="true" :file-list="fileList"
        :on-preview="handlePictureCardPreview" :on-remove="handleRemove" :on-success="handleAvatarSuccess"
        list-type="picture-card" method="post" preview-teleported>
        <el-icon>
          <Plus />
        </el-icon>
      </el-upload>
    </div>
    <el-form ref="itemFormRef" :model="itemForm" :rules="itemRules" label-width="120px">
      <el-row :gutter="20">
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="商品编号" prop="itemNum">
            <el-input v-model="itemForm.itemNum" readonly />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="商品名称" prop="itemName">
            <el-input v-model="itemForm.itemName" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="商品类型" prop="typeId">
            <el-input v-model="selectedTypeName" disabled placeholder="点击选择商品类型" />
            <el-button size="small" style="margin-left: 10px;" type="primary" @click="openTypeDialog">选择</el-button>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="库存数量" prop="store">
            <el-input v-model.number="itemForm.store" type="number" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="品牌" prop="brandId">
            <el-select v-model="itemForm.brandId" placeholder="请选择品牌">
              <el-option v-for="item in brandList" :key="item.brandId" :label="item.brandName" :value="item.brandId" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="门店" prop="storeId">
            <el-select v-model="itemForm.storeId" placeholder="请选择门店">
              <el-option v-for="item in storeList" :key="item.storeId" :label="item.storeName" :value="item.storeId" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="供应商" prop="supplyId">
            <el-select v-model="itemForm.supplyId" placeholder="请选择供应商">
              <el-option v-for="item in supplyList" :key="item.supplyId" :label="item.supplyName"
                :value="item.supplyId" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="产地" prop="placeId">
            <el-select v-model="itemForm.placeId" placeholder="请选择产地">
              <el-option v-for="item in placeList" :key="item.placeId" :label="item.placeName" :value="item.placeId" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="单位" prop="unitId">
            <el-select v-model="itemForm.unitId" placeholder="请选择单位">
              <el-option v-for="item in unitList" :key="item.unitId" :label="item.unitName" :value="item.unitId" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="进货价格" prop="price">
            <el-input v-model.number="itemForm.price" type="number" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="销售价格" prop="sellPrice">
            <el-input v-model.number="itemForm.sellPrice" type="number" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="会员价格" prop="vipPrice">
            <el-input v-model.number="itemForm.vipPrice" type="number" />
          </el-form-item>
        </el-col>
        <el-col :md="24" :sm="24" :xs="24">
          <el-form-item label="商品描述" prop="itemDesc">
            <el-input v-model="itemForm.itemDesc" :rows="3" type="textarea" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="生产日期" prop="itemDate">
            <el-date-picker v-model="itemForm.itemDate" placeholder="选择生产日期" type="date" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="到期日期" prop="endDate">
            <el-date-picker v-model="itemForm.endDate" placeholder="选择到期日期" type="date" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="促销标题" prop="hotTitle">
            <el-input v-model="itemForm.hotTitle" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="制造商" prop="facturer">
            <el-input v-model="itemForm.facturer" />
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="商品状态" prop="statue">
            <el-select v-model="itemForm.statue" placeholder="请选择状态">
              <el-option :value="0" label="上架" />
              <el-option :value="1" label="下架" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :md="8" :sm="12" :xs="24">
          <el-form-item label="创建者" prop="createBy">
            <el-input v-model="itemForm.createBy" />
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <el-button @click="dialogItemVisible = false">取消</el-button>
      <el-button type="primary" @click="submitItem">保存</el-button>
    </template>
  </el-dialog>

  <!-- 商品类型选择对话框 -->
  <el-dialog v-model="dialogTypeVisible" :width="dialogWidth" title="选择商品类型">
    <el-tree ref="typeTreeRef" :data="typeList" :expand-on-click-node="false" :highlight-current="true"
      :props="typeTreeConfig" default-expand-all node-key="id" @node-click="handleTypeNodeClick">
      <template #default="{ node }">
        <span>{{ node.label }}</span>
      </template>
    </el-tree>
    <template #footer>
      <el-button @click="dialogTypeVisible = false">取消</el-button>
      <el-button type="primary" @click="confirmTypeSelection">确认</el-button>
    </template>
  </el-dialog>

  <!-- 采购对话框 -->
  <el-dialog v-model="buyDialog" title="商品采购" :width="dialogWidth">
    <el-form :model="buyForm" label-width="120px" ref="buyFormRef">
      <el-form-item label="商品名称">
        <el-input v-model="buyForm.itemName" disabled />
      </el-form-item>
      <el-form-item label="门店">
        <el-input v-model="buyForm.storeName" disabled />
      </el-form-item>
      <el-form-item label="供应商">
        <el-input v-model="buyForm.supplyName" disabled />
      </el-form-item>
      <el-form-item label="产地">
        <el-input v-model="buyForm.placeName" disabled />
      </el-form-item>
      <el-form-item label="采购数量" prop="buyNum">
        <el-input v-model="buyForm.buyNum" type="number" />
      </el-form-item>
      <el-form-item label="采购人" prop="buyUser">
        <el-input v-model="buyForm.buyUser" />
      </el-form-item>
      <el-form-item label="联系电话" prop="phone">
        <el-input v-model="buyForm.phone" />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="buyDialog = false">取消</el-button>
      <el-button type="primary" @click="submitPurchase">确认采购</el-button>
    </template>
  </el-dialog>

  <!-- 图片预览组件 -->
  <el-image-viewer v-if="dialogVisible" :hide-on-click-modal="false" :initial-index="0" :teleported="true"
    :url-list="[dialogImageUrl]" :z-index="3000" :zoom-rate="1.2" @close="dialogVisible = false" />
  <!-- 出库对话框组件 -->
  <el-dialog v-model="itemOutDialog" width="60%">
    <h2>商品采购</h2>

    <el-form :model="outForm" label-width="120px">
      <el-form-item label="商品名称">
        {{ outForm.itemName }}
      </el-form-item>
      <el-form-item label="仓库">
        {{ outForm.storeName }}
      </el-form-item>
      <el-form-item label="商品库存">
        {{ outForm.store }}
      </el-form-item>
      <el-form-item label="出库数量">
        <el-input v-model="outForm.outNum" style="width: 80%" />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="saveOutOrder">保存</el-button>
        <el-button>取消</el-button>
      </el-form-item>
    </el-form>
  </el-dialog>
</template>

<script setup>
import { computed, nextTick, onMounted, reactive, ref } from "vue";
import { Plus } from '@element-plus/icons-vue';
import { ElMessage, ElMessageBox } from "element-plus";
import { categoryApi } from "@/api/category";
import { itemApi, uploadImageUrl } from "@/api/item";

// 处理图片加载错误
function handleImageError(index) {
  console.warn(`图片加载失败: ${index}`);
}

// 对话框状态和图片上传相关
const dialogItemVisible = ref(false);
const dialogTypeVisible = ref(false);
const dialogVisible = ref(false);
const dialogImageUrl = ref('');
const itemFormRef = ref(null);
const typeTreeRef = ref(null);
const searchFormRef = ref(null);
const fileList = ref([]);

// 采购表单数据
const buyForm = reactive({
  productId: '', storeId: '', supplyId: '', placeId: '', itemName: '',
  storeName: '', supplyName: '', placeName: '', buyNum: '', buyUser: '', phone: ''
});

// 采购对话框状态
const buyDialog = ref(false);
const buyFormRef = ref(null);

// 列表数据
const itemList = ref([]);
const total = ref(0);

// 响应式对话框宽度
const dialogWidth = computed(() => {
  return window.innerWidth < 768 ? '90%' : window.innerWidth < 1024 ? '70%' : '60%';
});

// 树形结构配置
const typeTreeConfig = {
  id: 'id',
  label: 'label',
  children: 'children'
};

// 查询表单数据
const searchForm = reactive({
  itemNum: '',
  itemName: '',
  statue: null,
});

// 商品表单数据
const initialItemFormState = {
  itemNum: '', itemName: '', typeId: null, store: 0, brandId: null,
  storeId: null, supplyId: null, placeId: null, unitId: null, price: 0,
  sellPrice: 0, vipPrice: 0, itemDesc: '', itemDate: null, endDate: null,
  hotTitle: '', facturer: '', statue: 1, imgs: [], createBy: '', id: null
};

const itemForm = reactive({ ...initialItemFormState });

// 显示选中的类型名称
const selectedTypeName = ref('');

// 表单验证规则
const itemRules = {
  itemNum: [{ required: true, message: '请输入商品编号', trigger: 'blur' }],
  itemName: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
  typeId: [{ required: true, message: '请选择商品类型', trigger: 'change' }],
  store: [{ required: true, message: '请输入库存数量', trigger: 'blur' }],
  brandId: [{ required: true, message: '请选择品牌', trigger: 'change' }],
  storeId: [{ required: true, message: '请选择门店', trigger: 'change' }],
  supplyId: [{ required: true, message: '请选择供应商', trigger: 'change' }],
  placeId: [{ required: true, message: '请选择产地', trigger: 'change' }],
  unitId: [{ required: true, message: '请选择单位', trigger: 'change' }],
  price: [{ required: true, message: '请输入进货价格', trigger: 'blur' }],
  sellPrice: [{ required: true, message: '请输入销售价格', trigger: 'blur' }],
  vipPrice: [{ required: true, message: '请输入会员价格', trigger: 'blur' }],
  itemDate: [{ required: true, message: '请选择生产日期', trigger: 'change' }],
  endDate: [{ required: true, message: '请选择到期日期', trigger: 'change' }],
  statue: [{ required: true, message: '请选择商品状态', trigger: 'change' }],
};

// 列表数据
const typeList = ref([]), supplyList = ref([]), placeList = ref([]), unitList = ref([]), brandList = ref([]), storeList = ref([]);

// 重置与商品表单相关的状态（如选中的类型名称和文件列表）
function resetItemRelatedState() {
  selectedTypeName.value = '';
  fileList.value = [];
}

// 打开商品信息对话框并加载所有数据
function openItemDialog() {
  // 重置表单和相关状态
  Object.assign(itemForm, { ...initialItemFormState });
  resetItemRelatedState();
  //确保DOM更新完毕后再调用resetFields，以保证itemFormRef可用
  nextTick(() => {
    if (itemFormRef.value) {
      itemFormRef.value.resetFields();
    }
  });

  // 获取商品编号
  itemApi.getItemCode()
    .then(response => {
      itemForm.itemNum = response.data;
    })
    .catch(() => {
      ElMessage.error('获取商品编号失败');
    });

  dialogItemVisible.value = true;
}

// 打开商品类型选择对话框
function openTypeDialog() {
  dialogTypeVisible.value = true;
}

// 处理类型树节点点击
function handleTypeNodeClick(data) {
  if (data.children && data.children.length > 0) {
    ElMessage.warning('只能选择叶子节点');
    typeTreeRef.value.setCurrentKey(null);
  } else {
    typeTreeRef.value.setCurrentKey(data.id);
  }
}

// 确认类型选择
function confirmTypeSelection() {
  const selectedNode = typeTreeRef.value.getCurrentNode();
  if (!selectedNode) {
    ElMessage.warning('请选择一个商品类型');
    return;
  }
  if (selectedNode.children && selectedNode.children.length > 0) {
    ElMessage.warning('只能选择叶子节点');
    return;
  }
  itemForm.typeId = selectedNode.id;
  selectedTypeName.value = selectedNode.label;
  dialogTypeVisible.value = false;
}

// 图片上传成功回调
function handleAvatarSuccess(response) {
  const imageUrl = typeof response === 'string' ? response : (response.url || '');
  if (imageUrl) {
    if (!Array.isArray(itemForm.imgs)) itemForm.imgs = [];
    itemForm.imgs.push(imageUrl);
    fileList.value.push({ url: imageUrl, status: 'success' });
  }
}

// 移除图片
function handleRemove(file) {
  const index = fileList.value.indexOf(file);
  if (index !== -1) {
    fileList.value.splice(index, 1);
    itemForm.imgs.splice(index, 1);
  }
}

// 图片预览
function handlePictureCardPreview(uploadFile) {
  dialogImageUrl.value = uploadFile.url;
  dialogVisible.value = true;
}

// 提交表单
function submitItem() {
  itemFormRef.value.validate((valid) => {
    if (valid) {
      const apiCall = itemForm.id ? itemApi.updateItem(itemForm) : itemApi.saveItem(itemForm);
      apiCall
        .then((response) => {
          if (response.data && response.data.code === 500) {
            ElMessage.error(response.data.message || '操作失败，请稍后重试');
            return;
          }
          ElMessage.success(itemForm.id ? '修改成功' : '添加成功');
          dialogItemVisible.value = false;
          Object.assign(itemForm, { ...initialItemFormState });
          resetItemRelatedState();
          loadItemList(1);
        })
        .catch((error) => {
          const errorMsg = error.response?.data?.message || '保存失败，请稍后重试';
          ElMessage.error(errorMsg);
        });
    }
  });
}

// 提交采购表单
function submitPurchase() {
  buyFormRef.value.validate((valid) => {
    if (valid) {
      itemApi.saveBuy(buyForm)
        .then((response) => {
          if (response.data.code === 200) {
            ElMessage.success(response.data.message || '采购成功');
            buyDialog.value = false;
            loadItemList(1); // 刷新商品列表
            Object.assign(buyForm, {
              productId: '', storeId: '', supplyId: '', placeId: '', itemName: '',
              storeName: '', supplyName: '', placeName: '', buyNum: '', buyUser: '', phone: ''
            });
          } else {
            ElMessage.error(response.data.message || '采购失败');
          }
        })
        .catch((error) => {
          console.error('采购失败:', error);
          ElMessage.error('采购失败，请稍后重试');
        });
    } else {
      ElMessage.warning('请填写完整的采购信息');
    }
  });
}

// 加载所有数据
function loadAllData() {
  Promise.all([
    categoryApi.getCategoryTree(),
    itemApi.getSupplyList(),
    itemApi.getPlaceList(),
    itemApi.getUnitList(),
    itemApi.getBrandList(),
    itemApi.getStoreList()
  ])
    .then(([typeRes, supplyRes, placeRes, unitRes, brandRes, storeRes]) => {
      typeList.value = typeRes.data;
      supplyList.value = supplyRes.data;
      placeList.value = placeRes.data;
      unitList.value = unitRes.data;
      brandList.value = brandRes.data;
      storeList.value = storeRes.data;
    })
    .catch(() => {
      ElMessage.error('加载数据失败，请稍后重试');
    });
}

// 加载商品列表
function loadItemList(pageNum = 1, pageSize = 10) {
  const params = {
    pageNum,
    pageSize,
    itemNum: searchForm.itemNum,
    itemName: searchForm.itemName,
    statue: searchForm.statue,
  };
  itemApi.getItemList(params)
    .then((response) => {
      // 确保从后端正确解析数据
      if (response.data && response.data.items) {
        itemList.value = response.data.items;
        total.value = response.data.total;
      } else {
        // 处理可能的空数据或错误格式
        itemList.value = [];
        total.value = 0;
        ElMessage.warning('商品数据格式不正确或为空');
      }
    })
    .catch(() => {
      ElMessage.error('加载商品列表失败');
    });
}

// 处理查询
function handleSearch() {
  loadItemList(1); // 查询时总是从第一页开始
}

// 重置查询表单
function resetSearchForm() {
  if (searchFormRef.value) {
    searchFormRef.value.resetFields();
  }
  // 手动清空searchForm reactive对象的值，因为resetFields可能不会完全清空
  searchForm.itemNum = '';
  searchForm.itemName = '';
  searchForm.statue = null;
  loadItemList(1); // 重置后重新加载数据
}

// 删除商品
function handleDeleteItem(id) {
  ElMessageBox.confirm('确定要删除这个商品吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  })
    .then(() => {
      itemApi.deleteItem(id)
        .then((response) => {
          if (response.data.code === 200) {
            ElMessage.success(response.data.message || '删除成功');
            loadItemList(1);
          } else {
            ElMessage.error(response.data.message || '删除失败');
          }
        })
        .catch(() => {
          ElMessage.error('删除失败');
        });
    })
    .catch(() => {
      ElMessage.info('已取消删除');
    });
}

// 处理商品下架
function handleDownItem(id) {
  itemApi.downItem(id)
    .then((response) => {
      if (response.data.code === 200) {
        ElMessage.success(response.data.message || '下架成功');
        loadItemList(1);
      } else {
        ElMessage.error(response.data.message || '下架失败');
      }
    })
    .catch(() => {
      ElMessage.error('下架失败');
    });
}

// 处理商品上架
function handleUpItem(id) {
  itemApi.upItem(id)
    .then((response) => {
      if (response.data.code === 200) {
        ElMessage.success(response.data.message || '上架成功');
        loadItemList(1);
      } else {
        ElMessage.error(response.data.message || '上架失败');
      }
    })
    .catch(() => {
      ElMessage.error('上架失败');
    });
}

// 打开修改对话框
function openUpdateDialog(row) {
  dialogItemVisible.value = true;
  Object.assign(itemForm, {
    id: row.id,
    itemNum: row.itemNum || '',
    itemName: row.itemName || '',
    typeId: row.typeId,
    store: row.store || 0,
    brandId: row.brandId,
    storeId: row.storeId,
    supplyId: row.supplyId,
    placeId: row.placeId,
    unitId: row.unitId,
    price: row.price || 0,
    sellPrice: row.sellPrice || 0,
    vipPrice: row.vipPrice || 0,
    itemDesc: row.itemDesc || '',
    itemDate: row.itemDate,
    endDate: row.endDate,
    hotTitle: row.hotTitle || '',
    facturer: row.facturer || '',
    statue: row.statue,
    imgs: row.imgs || [],
    createBy: row.createBy || ''
  });
  fileList.value = (row.imgs || []).map(url => ({ url, status: 'success' }));
  selectedTypeName.value = row.cateName || ''; // 使用 cateName 对应后端的商品类型名称
  loadAllData();
}

// 生命周期钩子
onMounted(() => {
  loadItemList(1);
  loadAllData(); // 组件挂载时加载所有下拉列表数据
});

// 处理分页
function handlePageChange(value) {
  loadItemList(value);
}

// 打开采购对话框
function openPurchaseDialog(row) {
  buyDialog.value = true;
  // 发送ajax请求，获取需要带入的数据
  itemApi.getBuyAutoInfo(row.id)
    .then((response) => {
      // 获取响应数据对象
      const item = response.data;
      // 将响应数据赋值给buyForm表单
      buyForm.productId = item.id;
      buyForm.itemName = item.itemName;
      buyForm.storeId = item.storeId;
      buyForm.storeName = item.storeName;
      buyForm.supplyId = item.supplyId;
      buyForm.supplyName = item.supplyName;
      buyForm.placeId = item.placeId;
      buyForm.placeName = item.placeName;
    })
    .catch((error) => {
      console.error('获取采购信息失败:', error);
      ElMessage.error('获取采购信息失败');
    });
}
//声明出库对话框状态
const itemOutDialog = ref(false);
//声明商品出库form表单
const outForm = reactive({
  itemName: '',
  storeName: '',
  store: 0,
  outNum: 0,
  productId: undefined
});

//定义函数打开商品出库对话框
function openOutDialog(row) {
  itemOutDialog.value = true;
  outForm.itemName = row.itemName;
  outForm.store = row.store;
  outForm.storeName = row.storeName;
  outForm.productId = row.id;
  outForm.outNum = 0;
}
//定义函数发生商品出库请求
function saveOutOrder() {
  if (!outForm.outNum || outForm.outNum <= 0) {
    ElMessage.warning('请输入正确的出库数量');
    return;
  }
  if (outForm.outNum > outForm.store) {
    ElMessage.warning('出库数量不能大于库存');
    return;
  }
  itemApi.doItemOutStore(outForm)
    .then((response) => {
      if (response.data.code == 200) {
        itemOutDialog.value = false;
        loadItemList(1); // 出库成功后刷新商品列表
      }
      ElMessage(response.data.message);
    })
    .catch((error) => {
      console.log(error);
    })
}
</script>

<style scoped></style>