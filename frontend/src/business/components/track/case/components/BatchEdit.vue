<template>
  <div>
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="25%"
      class="batch-edit-dialog"
      :destroy-on-close="true"
      @close="handleClose"
    >
      <el-form :model="form" label-position="right" label-width="150px" size="medium" ref="form" :rules="rules">
        <el-form-item :label="$t('test_track.case.batch_update', [size])" prop="type">
          <el-select v-model="form.type" style="width: 80%" @change="changeType">
            <el-option v-for="(type, index) in typeArr" :key="index" :value="type.id" :label="type.name"/>
          </el-select>
        </el-form-item>
        <el-form-item  v-if="form.type === 'projectEnv'" :label="$t('test_track.case.updated_attr_value')">
          <env-popover :env-map="projectEnvMap" :project-ids="projectIds" @setProjectEnvMap="setProjectEnvMap"
                       :project-list="projectList" ref="envPopover"/>
        </el-form-item>
        <el-form-item v-else :label="$t('test_track.case.updated_attr_value')" prop="value">
          <el-select v-model="form.value" style="width: 80%" :filterable="filterable">
            <el-option v-for="(option, index) in options" :key="index" :value="option.id" :label="option.name">
              <div v-if="option.email">
                <span>{{option.id}}({{option.name}})</span>
              </div>
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <ms-dialog-footer
          @cancel="dialogVisible = false"
          @confirm="submit('form')"/>
      </template>
    </el-dialog>
  </div>
</template>

<script>
  import MsDialogFooter from "../../../common/components/MsDialogFooter";
  import {listenGoBack, removeGoBackListener} from "@/common/js/utils";
  import EnvPopover from "@/business/components/api/automation/scenario/EnvPopover";

  export default {
    name: "BatchEdit",
    components: {
      EnvPopover,
      MsDialogFooter
    },
    props: {
      typeArr: Array,
      valueArr: Object,
      dialogTitle: {
        type: String,
        default() {
          return this.$t('test_track.case.batch_operate')
        }
      },
    },
    data() {
      return {
        dialogVisible: false,
        form: {},
        size: 0,
        rules: {
          type: {required: true, message: this.$t('test_track.case.please_select_attr'), trigger: ['blur','change']},
          value: {required: true, message: this.$t('test_track.case.please_select_attr_value'), trigger: ['blur','change']}
        },
        options: [],
        filterable: false,
        projectList: [],
        projectIds: new Set(),
        selectRows: new Set(),
        projectEnvMap: new Map()
      }
    },
    methods: {
      submit(form) {
        this.$refs[form].validate((valid) => {
          if (valid) {
            this.form.projectEnvMap = this.projectEnvMap;
            if (this.form.type === 'projectEnv') {
              if (!this.$refs.envPopover.checkEnv()) {
                return false;
              }
            }
            this.$emit("batchEdit", this.form);
            this.dialogVisible = false;
          } else {
            return false;
          }
        });
      },
      setProjectEnvMap(projectEnvMap) {
        this.projectEnvMap = projectEnvMap;
      },
      open(size) {
        this.dialogVisible = true;
        if (size) {
          this.size = size;
        } else {
          this.size = this.$parent.selectDataCounts;
        }
        listenGoBack(this.handleClose);
        this.getWsProjects();
      },
      setSelectRows(rows) {
        this.selectRows = rows;
        this.projectIds.clear();
        this.selectRows.forEach(row => {
          this.projectIds.add(row.projectId)
        })
      },
      handleClose() {
        this.form = {};
        this.options = [];
        removeGoBackListener(this.handleClose);
      },
      changeType(val) {
        this.$set(this.form, "value", "");
        this.filterable = val === "maintainer" || val === "executor";
        this.options = this.valueArr[val];
        this.typeArr.forEach(item => {
          if (item.id === val) {
            if (item.optionMethod) {
              this.options = [];
              item.optionMethod(this.options);
            }
            return;
          }
        });
      },
      getWsProjects() {
        this.$get("/project/listAll", res => {
          this.projectList = res.data;
        })
      },
    }
  }
</script>

<style scoped>

</style>
