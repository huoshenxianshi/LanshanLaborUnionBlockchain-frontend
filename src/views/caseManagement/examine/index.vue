<template>
  <div class="app-container">
    <div class="input">
      <el-input v-model="searchText" placeholder="请输入案件号搜索案件" size="medium" class="searchCase">
        <el-button slot="append" type="primary" icon="el-icon-search" @click="search()">搜索</el-button>
      </el-input>
    </div>
    <div class="content">
      <avue-crud
        v-model="obj"
        :data="allCase"
        :option="option"
        :page="page"
        @on-load="onLoad"
      >
        <template slot="menuLeft">
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="middle"
            plain
            @click.stop="handleAdd()"
          >新增</el-button>
        </template>
        <template slot="menu" slot-scope="scope">
          <el-button
            type="text"
            icon="el-icon-edit"
            size="small"
            plain
            @click.stop="handleEdit(scope.row,scope.index)"
          >编辑</el-button>
          <el-button
            type="text"
            icon="el-icon-view"
            size="small"
            plain
            @click.stop="handleView(scope.row,scope.index)"
          >详情</el-button>
          <el-button
            type="text"
            icon="el-icon-cpu"
            size="small"
            plain
            @click.stop="handleTrace(scope.row,scope.index)"
          >区块链追溯</el-button>
        </template>
      </avue-crud>
      <el-dialog title="案件表单" :visible.sync="dialogFormVisible" width="80%">
        <!-- <el-tabs v-model="activeName" type="card">
          <el-tab-pane label="案件表单" name="first"> -->
        <avue-form ref="form" v-model="infoData" :option="infoOption">
          <template slot="menuForm">
            <el-button type="primary" @click="handleSubmit">提 交</el-button>
          </template>
        </avue-form>
        <!-- </el-tab-pane> -->
        <!-- <el-tab-pane label="案件要素表" name="second">
            <avue-form
              ref="form"
              v-model="laborData"
              :option="infolaborOption"
            >
              <template slot="menuForm">
                <el-button type="primary" @click="handleSubmit">提 交</el-button>
              </template>
            </avue-form>
          </el-tab-pane> -->
        <!-- </el-tabs> -->
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { getAllCase, getCase, modifyForm } from "@/api/case";
import Option from "../newcase/newcase/elements";
import laborOption from "@/components/Componentform/option";
export default {
  components: {},
  data() {
    return {
      page: {
        pageSize: 10
      },
      obj: {},
      activeName: "first",
      dialogFormVisible: false,
      infoOption: Option.option,
      infolaborOption: laborOption.option,
      laborData: {},
      infoData: {},
      searchText: "",
      allCase: [],
      id: "",
      option: {
        align: "center",
        menuAlign: "center",
        // viewBtn: true,
        editBtn: false,
        addBtn: false,
        delBtn: false,
        column: [
          {
            label: "案件号码",
            prop: "case_id"
          },
          {
            label: "申诉人姓名",
            prop: "applicant_name"
          },
          {
            label: "创建时间",
            prop: "created_at"
          },
          {
            label: "调解事项",
            prop: "title"
          },
          {
            label: "案件进度更新时间",
            prop: "updated_at"
          },
          {
            label: "用人单位",
            prop: "respondent_name"
          }
        ]
      }
    };
  },
  created() {
  },
  methods: {
    async onLoad(page) {
      const text = await getAllCase({
        currentPage: page.currentPage,
        pageSize: page.pageSize
      });
      this.page = page
      this.page.total = text.data.total_count
      this.allCase = text.data.list;
    },
    async handleEdit(row, index) {
      this.dialogFormVisible = true;
      const text = await getCase({ caseId: this.allCase[index].case_id });
      this.id = this.allCase[index].id;
      const infoData = { title: text.data.title, content: text.data.content, category_id: text.data.category.id }
      Object.assign(this.infoData, infoData, text.data.applicant, text.data.respondent, text.data.materials)
      this.laborData = text.data.form;
    },
    handleTrace(row, index) {
      this.$router.push({
        name: "CasetraceView",
        params: {
          caseId: row.case_id
        }
      });
    },
    async handleSubmit() {
      const upload = {
        form_id: this.laborData.id,
        applicant: {
          applicant_name: this.infoData.applicant_name.trim(),
          applicant_birth: this.infoData.applicant_birth,
          applicant_nationality: this.infoData.applicant_nationality.trim(),
          applicant_id: this.infoData.applicant_id.trim(),
          applicant_contact: this.infoData.applicant_contact.trim(),
          applicant_address: this.infoData.applicant_address.trim()
        },
        category_id: this.infoData.category_id,
        content: this.infoData.content.trim(),
        materials: this.infoData.materials,
        respondent: {
          employer_name: this.infoData.employer_name.trim(),
          employer_faren: this.infoData.employer_faren.trim(),
          employer_uscc: this.infoData.employer_uscc.trim(),
          employer_contact: this.infoData.employer_contact.trim(),
          employer_address: this.infoData.employer_address.trim()
        },
        title: this.infoData.title.trim()
      };
      const res = await modifyForm({ id: this.id, data: upload });
      if (res.message === "success") {
        this.dialogFormVisible = false;
      } else {
        this.$message({
          message: "修改失败",
          type: "error"
        });
      }
    },
    handleView(row, index) {
      this.$router.push("caseManagement/view/" + row.id);
    },
    handleAdd() {
      this.$router.push("/newcase/newform");
    },
    async search() {
      await getCase({ caseId: this.searchText })
        .then(res => {
          this.allCase = [
            {
              applicant_name: "",
              case_id: "",
              title: "",
              updated_at: "",
              respondent_name: ""
            }
          ];
          this.allCase[0].applicant_name = res.data.applicant.applicant_name;
          this.allCase[0].case_id = res.data.case_id;
          this.allCase[0].title = res.data.title;
          this.allCase[0].updated_at = res.data.updated_at;
          this.allCase[0].respondent_name = res.data.respondent.employer_name;
        })
        .catch(err => {
          alert("未查到该案件的信息");
          console.log(err);
          this.text();
        });
    }
  }
};
</script>

<style lang="less" scoped>
.app-container {
  .input {
    width: 500px;
  }
  .content {
    margin-top: 30px;
    // background-color: rgb(243,243,243);
  }
}
</style>
