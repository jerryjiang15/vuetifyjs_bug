<template>
  <div>
    <DepartmentSelectDialog 
    v-if="initDepartmentSelectDialog" 
    :title="$t('GENERAL.SELECT_DEPARTMENT')"
    v-on:close="initDepartmentSelectDialog = false"
    v-on:save="saveDepartmentSelectDialog"
    :besideID="besideID"
    >
    </DepartmentSelectDialog>
    <div class="row">
      <v-snackbar
        v-model="snackbar"
        :top="true"
        :color="snackbarColor"
      >
        <b>{{ snackbarMsg }}</b>
        <v-btn
          dark
          text
          @click="snackbar = false"
        >
          {{ $t("GENERAL.CLOSE") }}
      </v-btn>
      </v-snackbar>
      <div class="col-md-12">
        <v-alert
          v-if="systemError != ''"
          border="right"
          colored-border
          type="error"
          elevation="2">
          {{ systemError }}
        </v-alert>
        <KTPortlet v-bind:title="$t('ROLE.DEPARTMENT_1')">
          <template v-slot:toolbar>
            <v-btn color="primary" @click="newItem()">{{ $t("ROLE.DEPARTMENT_2") }}</v-btn>
          </template>
          <template v-slot:body>
            <v-row
            class="pa-4"
            justify="space-between"
            >
                <v-col cols="5">
                    <v-treeview
                        :active.sync="active"
                        :items="items"
                        :open.sync="open"
                        activatable
                        color="primary"
                        transition
                        ref="treeview"
                        hoverable
                        v-on:update:active="onActive"
                    >
                    <template v-slot:label="{ item }">
                        <v-icon v-if="loading" small>fas fa-circle-notch fa-spin</v-icon> <b>{{ item.name }}</b>
                    </template>
                    </v-treeview>
                </v-col>

                <v-divider vertical></v-divider>

                <v-col
                    class="d-flex text-center"
                >
                    <v-scroll-y-transition mode="out-in">
                        <div
                        v-if="showFromBox == false"
                        class="title grey--text text--lighten-1 font-weight-light col-md-12"
                        style="align-self: center;align: center;"
                        >
                        {{ $t("ROLE.DEPARTMENT_4") }}
                        </div>
                        <div
                        v-else
                        :key="editedItem.id"
                        class="col-md-12 mx-auto"
                        >
                        <v-row align="center" justify="center">
                            <h5>{{ submitType == 1 ? $t("ROLE.DEPARTMENT_2") : $t("ROLE.DEPARTMENT_3") }}</h5>
                        </v-row>
                        <v-row>
                            <v-form ref="form" class="col-md-12">
                                <v-col cols="12" sm="12" md="12">
                                    <v-text-field 
                                    v-model="editedItem.parent.name"
                                    :label="$t('ROLE.DEPARTMENT_5')"
                                    readonly
                                    :rules="requiredRules"
                                    v-on:keyup.enter="save"
                                    v-on:click="openDepartmentSelectDialog"
                                    :clearable="editedItem.parent.id > 0 ? true : false"
                                    ></v-text-field>
                                </v-col>
                                <v-col cols="12" sm="12" md="12">
                                    <v-text-field 
                                    v-model="editedItem.name"
                                    :label="$t('ROLE.DEPARTMENT_6')"
                                    :rules="nameRules"
                                    v-on:keyup.enter="save"
                                    ></v-text-field>
                                </v-col>
                            </v-form>
                        </v-row>
                        <v-row align="center" justify="center">
                            <v-btn :loading="saveBtnLoading" @click="save" color="blue darken-1" dark>{{ $t("GENERAL.SAVE") }}</v-btn>
                        </v-row>
                        </div>
                    </v-scroll-y-transition>
                </v-col>
            </v-row>
          </template>
        </KTPortlet>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import KTPortlet from "@/views/partials/content/Portlet.vue";
import { SET_BREADCRUMB } from "@/store/breadcrumbs.module";
import { OPERATION_ACCOUNT_DATA } from "@/store/data.module";
import { SYS_CMD } from "@/common/config";
import DepartmentSelectDialog from "@/views/partials/content/DepartmentSelectDialog.vue";

// const pause = ms => new Promise(resolve => setTimeout(resolve, ms))

export default {
  data() {
    return {
        initDepartmentSelectDialog: false,
        besideID: 0,
        submitType: 2,  //1为新增， 2为编辑
        showFromBox: false,
        defaultItem: {id: 0, name: "", parent: {id: 0, name: this.$t("GENERAL.NONE")}},
        editedItem: {id: 0, name: "", parent: {id: 0, name: ""}},
        saveBtnLoading: false,
        snackbar: false,
        snackbarColor: "success",
        snackbarMsg: "",
        systemError: "",
        loading: true,
        active: [],
        open: [],
        selectList: [],
        requiredRules: [
            v => v != "" || this.$t("AUTH.VALIDATION.REQUIRED_3", { name: ""}),
        ],
        nameRules: [
            v => !!v || this.$t("AUTH.VALIDATION.REQUIRED_2", { name: "", num: 2}),
            v => (v && v.length >= 2) || this.$t("AUTH.VALIDATION.REQUIRED_2", { name: "", num: 2})
        ],
        items: [
            {
                id: 0,
                name: this.$t("GENERAL.LOADING_TEXT"),
            }
        ]
    };
  },
  components: {
    KTPortlet,
    DepartmentSelectDialog
  },
  watch: {
    'editedItem.parent.name' : function (val) {
      if (val == null || val == "")
      {
        this.editedItem.parent.id = 0;
        this.editedItem.parent.name = this.$t("GENERAL.NONE");
      }
    }
  },
  methods: {
    openDepartmentSelectDialog () {
        // this.clientBesideID = this.editedItem.client_id;
        this.initDepartmentSelectDialog = true;
    },
    saveDepartmentSelectDialog(item) 
    {
        this.initDepartmentSelectDialog = false;

        this.editedItem.parent.id = item.id;
        this.editedItem.parent.name = item.name;
    },
    showSuccessBox () {
      this.snackbar = true;
      this.snackbarColor = "success";
      this.snackbarMsg = this.$t("GENERAL.SAVE_SUCCESS");
    },
    showFailBox () {
      this.snackbar = true;
      this.snackbarColor = "error";
      this.snackbarMsg = this.$t("GENERAL.SAVE_FAIL");
    },
    onActive(selects)
    { 
        Object.assign(this.editedItem, this.defaultItem);

        if (selects.length == 0)
        {
            this.showFromBox = false;
            return;
        }

        this.showFromBox = true;
        this.submitType = 2;

        var id = selects[0];

        this.besideID = id;
        
        this.getEditItem(id, this.items, {id: 0, name: this.$t("GENERAL.NONE")});
    },
    getEditItem(id, resData, parent)
    {
        if (this.editedItem.id > 0) return;

        for (const item of resData)
        {
            if (item.id == id) Object.assign(this.editedItem, {parent: parent, ...item});

            if (item.children) this.getEditItem(id, item.children, Object.assign({}, item));
        }
    },
    newItem ()
    {
        this.submitType = 1;
        this.showFromBox = true;

        this.besideID = 0;

        Object.assign(this.editedItem, this.defaultItem);
    },
    save () {

      if (this.$refs.form.validate()) {   //如果通过验证

        this.saveBtnLoading = true;

        if (this.submitType == 2) {  //如果是编辑
          this.$store
          .dispatch(OPERATION_ACCOUNT_DATA, 
          {
              department_id: this.editedItem.id,
              new_upper_department_id: this.editedItem.parent.id,
              department_name: this.editedItem.name,
              cmd: SYS_CMD.DEPARTMENT_MODIFY
          })
          .then(() => {  //获取成功
            this.getDataFromApi();
            this.showSuccessBox();
            this.saveBtnLoading = false;
          })
          .catch((err) => {  //获取失败
            this.snackbarMsg = err.errors[0];
            this.showFailBox();
            this.saveBtnLoading = false;
          });
        } 
        else 
        {  //如果是新增
          this.$store
          .dispatch(OPERATION_ACCOUNT_DATA, 
          {
              upper_department_id: this.editedItem.parent.id,
              department_name: this.editedItem.name,
              cmd: SYS_CMD.DEPARTMENT_CREATE
          })
          .then(() => {  //获取成功
            this.getDataFromApi();
            this.showSuccessBox();
            this.saveBtnLoading = false;
            Object.assign(this.editedItem, this.defaultItem);
            this.$refs.form.resetValidation();
          })
          .catch((err) => {  //获取失败
            this.snackbarMsg = err.errors[0];
            this.showFailBox();
            this.saveBtnLoading = false;
          });
        }
        
      }
      else
      {
        return false;
      }
    },
    formatData (array) {
        var data = [];

        for (const item of array)
        {
            var tmp = {
                id: item.department_id,
                name: item.department_name,
                children: this.formatData(item.child_array)
            }

            data.push(tmp);
        }

        return data;
    },
    getDataFromApi () {
      //读取数据
      this.$store
      .dispatch(OPERATION_ACCOUNT_DATA, 
      {
          root_department_id: 0,
          cmd: SYS_CMD.DEPARTMENT_STRUCTURE
      })
      .then(({ param }) => {  //获取成功
          this.items = this.formatData(param.department_root_array);
          this.loading = false;
      })
      .catch((err) => {  //获取失败
          this.systemError = err.errors[0];
      });
    }
  },
  computed: {
    ...mapState({
      errors: state => state.data.errors
    })
  },
  async mounted() {   //页面初始化完成
    
    //设置导航
    this.$store.dispatch(SET_BREADCRUMB, [
      { title: this.$t("MENU.ROLE_MANAGE") },
      { title: this.$t("MENU.ROLE_MANAGE_5") },
      { title: this.$t("MENU.ROLE_MANAGE_6"), route: "department" }
    ]);

    this.getDataFromApi();
  }
};
</script>
