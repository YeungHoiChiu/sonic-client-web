<script setup>
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { useI18n } from 'vue-i18n';
import PublicStepUpdate from '../components/PublicStepUpdate.vue';
import Pageable from '../components/Pageable.vue';
import axios from '../http/axios';
import ChildStepListView from '../components/ChildStepListView.vue';

const { t: $t } = useI18n();

const route = useRoute();
const dialogVisible = ref(false);
const pageData = ref({});
const pageSize = ref(15);
const pageCurrNum = ref(1);
const publicStepId = ref(0);
watch(dialogVisible, (newValue, oldValue) => {
  if (!newValue) {
    publicStepId.value = 0;
  }
});
const editPublicStep = async (id) => {
  publicStepId.value = id;
  await open();
};
const open = () => {
  dialogVisible.value = true;
};
const flush = (e) => {
  if (e) {
    dialogVisible.value = false;
  }
  getPublicStepList();
};
const getPublicStepList = (pageNum, pSize) => {
  pageSize.value = pSize || pageSize.value;
  pageCurrNum.value = pageNum || pageCurrNum.value;
  axios
    .get('/controller/publicSteps/list', {
      params: {
        projectId: route.params.projectId,
        page: pageCurrNum.value,
        pageSize: pageSize.value,
      },
    })
    .then((resp) => {
      pageData.value = resp.data;
    });
};
// 复制该公共步骤
// const copyPublicStepId = (id) => {
//   axios.get("/controller/publicSteps/copy", {
//     params: {
//       id
//     }
//   }).then(resp => {
//     if (resp['code'] === 2000) {
//       ElMessage.success({
//         message: resp['message']
//       });
//       getPublicStepList()
//     }
//   })
// }
const deletePublicStep = (id) => {
  axios
    .delete('/controller/publicSteps', {
      params: {
        id,
      },
    })
    .then((resp) => {
      if (resp.code === 2000) {
        ElMessage.success({
          message: resp.message,
        });
        getPublicStepList();
      }
    });
};
const curShowPublicStep = ref({
  id: null,
  projectId: route.params.projectId,
  platform: 0,
  name: '',
  steps: [],
});
/**
 * 点击列表上的某个公共元素信息之后，再拉取单个公共步骤的子步骤信息
 * @param id 公共步骤id
 */
const getPublicStepInfo = (id) => {
  curShowPublicStep.value.steps = [];
  axios.get('/controller/publicSteps', { params: { id } }).then((resp) => {
    if (resp.code === 2000) {
      curShowPublicStep.value = resp.data;
    }
  });
};
onMounted(() => {
  getPublicStepList();
});
</script>

<template>
  <el-dialog
    v-model="dialogVisible"
    :title="$t('publicStepTS.info')"
    width="70%"
  >
    <public-step-update
      v-if="dialogVisible"
      :public-step-id="publicStepId"
      :project-id="route.params.projectId"
      @flush="flush"
    />
  </el-dialog>
  <el-button size="mini" round type="primary" @click="open">{{
    $t('publicStepTS.add')
  }}</el-button>
  <el-table :data="pageData['content']" border style="margin-top: 10px">
    <el-table-column width="100" label="id" prop="id" align="center" />
    <el-table-column
      :label="$t('publicStepTS.name')"
      prop="name"
      header-align="center"
    />
    <el-table-column
      :label="$t('publicStepTS.platform')"
      width="110"
      align="center"
    >
      <template #default="scope">
        {{ scope.row.platform === 1 ? $t('publicStepTS.android') : 'iOS' }}
      </template>
    </el-table-column>
    <el-table-column
      :label="$t('publicStepTS.list')"
      width="130"
      align="center"
    >
      <template #default="scope">
        <el-popover
          placement="left"
          popper-class="public-step-popover"
          :fallback-placements="['bottom', 'top', 'right', 'left']"
          :width="700"
          trigger="click"
        >
          <child-step-list-view :steps="curShowPublicStep.steps" />
          <template #reference>
            <el-button size="mini" @click="getPublicStepInfo(scope.row.id)">{{
              $t('publicStepTS.viewSteps')
            }}</el-button>
          </template>
        </el-popover>
      </template>
    </el-table-column>
    <el-table-column :label="$t('common.operate')" width="250" align="center">
      <template #default="scope">
        <!--        <el-button type="primary"-->
        <!--                   size="mini"-->
        <!--                   v-on:click="copyPublicStepId(scope.row.id)"-->
        <!--        >-->
        <!--          复制-->
        <!--        </el-button>-->

        <el-button
          type="primary"
          size="mini"
          @click="editPublicStep(scope.row.id)"
        >
          {{ $t('common.edit') }}
        </el-button>

        <el-popconfirm
          style="margin-left: 10px"
          :confirm-button-text="$t('form.confirm')"
          :cancel-button-text="$t('form.cancel')"
          icon="el-icon-warning"
          icon-color="red"
          :title="$t('publicStepTS.sureDel')"
          @confirm="deletePublicStep(scope.row.id)"
        >
          <template #reference>
            <el-button type="danger" size="mini">
              {{ $t('common.delete') }}
            </el-button>
          </template>
        </el-popconfirm>
      </template>
    </el-table-column>
  </el-table>
  <pageable
    :is-page-set="true"
    :total="pageData['totalElements']"
    :current-page="pageData['number'] + 1"
    :page-size="pageData['size']"
    @change="getPublicStepList"
  ></pageable>
</template>
