<template>
  <div class="page_content_holder">
    <div class="padding_content">
      <div class="white_boxed_content lists_page">
        <header class="sec_header">
          <div class="align_left">
            <strong> Bounced Users</strong>
          </div>
          <input id="table_search" name="table_search" placeholder="Search here..."
                 type="search"
                 v-model="companyUserObj.search"
                 v-on:change="companySearch">
          <!--<div class="align_right">
              <strong class="actions_droper">Actions <i class="fas fa-sort-down"></i></strong>
              <ul class="actions_drop list_none">
                  <li>
                      <router-link to="/dashboard/attribute-listing"> Other Attributes Data</router-link>
                  </li>
                  <li v-if="export_button"><a v-on:click="exportUsers">Export</a></li>
                  <li v-if="export_file"><a v-on:click="downloadExportUsers">Download</a></li>
              </ul>
          </div>-->
        </header>
        <div class="list_holder">
          <div class="align_left mCustomScrollbar _mCS_1 mCS_no_scrollbar">
            <div class="side_widget">
              <strong class="title colored">Filter(s):</strong>
              <ul class="list_none side_tags">
                <li>
                  <a class="opener anchor_pointer">Status <i class="fas fa-sort-down"></i></a>
                  <ul class="list_none inner_tags side_tags indicators" style="width: 150px">
                    <li v-bind:class="dataTable.lookupProp == label ? 'active': ''"
                        v-for="(label, index) in actives">
                      <a class="sideBar-filter" style="cursor: pointer" v-bind:id="'active'+index"
                         v-on:click="fn_userFilter(label, index)">{{label}}
                        <i aria-hidden="true" class="fa fa-times"
                           v-if="dataTable.lookupProp == label"
                           v-on:click.stop="removeActive(index)"></i>

                      </a>
                    </li>
                  </ul>
                </li>
              </ul>
            </div>
          </div>
          <div class="table_holder mCustomScrollbar" style="padding-bottom: 60px;">
            <div class="dataTables_wrapper no-footer" id="example_wrapper">
              <listing v-bind:apiPayLoad="dataTable.payLoad"
                       v-bind:componentName="dataTable.componentName"
                       v-bind:footerComponent="dataTable.isFooterComponent"
                       v-bind:tableColumn="dataTable.column"
                       v-bind:tableColumnsClasses="dataTable.columnsClasses"
                       v-bind:tableFilterable="dataTable.filterable"
                       v-bind:tableHeadings="dataTable.headings"
                       v-bind:tableMapColumns="dataTable.mapColumns"
                       v-bind:tableReloading="dataTable.reloadProp"
                       v-bind:tableSearch="dataTable.searchProp"
                       v-bind:tableSideBarSearch="dataTable.lookupProp"
                       v-bind:tableSortable="dataTable.sortable">
                <template slot="listing" slot-scope="data">
                  <i class="fas fa-cog" v-on:click.stop="segmentAction(data.actionlisting)"></i>
                  <i class="fas fa-sort-down" v-on:click.stop="segmentAction(data.actionlisting)"></i>
                  <ul class="table_drop list_none"
                      v-bind:id="data.actionlisting.row.id+'_action'">
                    <li>
                      <router-link
                          v-bind:to="'/dashboard/user-stats/' + data.actionlisting.row.row_id">
                        <i class="fas fa-street-view"></i> View
                      </router-link>
                    </li>
                    <!--
                                                            <li v-if="data.actionlisting.row.status == 1">
                                                                <a v-on:click="userStatusUpdate(data.actionlisting.row.row_id,0)"><i
                                                                        class="fas fa-trash"></i> Inactive </a>
                                                            </li>
                                                            <li v-else>
                                                                <a v-on:click="userStatusUpdate(data.actionlisting.row.row_id,1)"><i
                                                                        class="fas fa-trash"></i> Active </a>
                                                            </li> -->
                    <li>
                      <a v-on:click="deleteUsers(data.actionlisting.row.id)"><i
                          class="fas fa-trash"></i> Delete </a>
                    </li>
                  </ul>
                </template>
                <template slot="image_url" slot-scope="data">
                  <div v-if="data.imagelisting.row.image_url">
                    <img v-bind:src="data.imagelisting.row.image_url"/>
                  </div>
                  <div v-else>
                    <img src="../../../../assets/images/profile_placeholder.png"/>
                  </div>
                </template>
                <template slot="user_id" slot-scope="data">
                  <b>User Id : {{data.userList.row.user_id}}</b><br/>
                  <b>Row Id : {{data.userList.row.row_id}}</b><br/>
                  <b>User Name : {{data.userList.row.username != null ? data.userList.row.username :
                      'N/A'}}</b><br/>
                  <span data-toggle="tooltip"
                        title="This user could not be able to receive notifications!" v-show="data.userList.row.is_revoked === 1"
                        class="label label-danger">Revoked</span>
                </template>
              </listing>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import listing from '../../../otherComponents/datatable/datatableComponent';
export default {
  name: "LookupListing",
  components: {
    listing,
  },
  data() {
    return {
      dataTable: {
        reloadProp: false,
        // column: ['image_url', 'user_id', 'email', 'device_type', 'last_login', 'is_active', 'Action'],
        column: ['row_id', 'bounce_email', 'action', 'status', 'code', 'message_id', 'bounce_date', 'Action'],
        // mapColumns: ['image_url', 'user_id', 'row_id', 'username', 'is_revoked', 'email', 'device_type', 'last_login', 'is_active', 'status', 'Action'],
        mapColumns: ['id', 'row_id', 'bounce_email', 'action', 'status', 'code', 'message_id', 'bounce_date', 'Action'],
        // filterable: ['user_id', 'email', 'last_login', 'device_type', 'is_active', 'status'],
        filterable: ['user_id', 'email', 'last_login', 'is_active'],
        // sortable: ['user_id', 'email', 'last_login', 'device_type', 'is_active', 'status'],
        sortable: ['user_id', 'email', 'last_login', 'is_active'],
        isFooterComponent: true,
        componentName: "companyUserListing",
        headings: {
          row_id: 'Row Id',
          bounce_email: 'Bounce Email',
          action: 'Action',
          status: 'Status',
          code: 'Code',
          message_id: 'Message Id',
          bounce_date: 'Bounced At'
        },
        columnsClasses: {
          row_id: 'width_10',
          bounce_email: 'width_20',
          action: 'width_10',
          status: 'width_10',
          code: 'width_20',
          message_id: 'width_20',
          bounce_date: 'width_15',
          Action: 'width_5'
        },
        payLoad: {
          resource: "bounced/users",
          action: "get",
        },
        searchProp: '',
        lookupProp: {},
      },
      companyUserObj: {
        search: '',
        lookupFilter: []
      },
      companyParamObj: {
        companyid: '0',
        name: '',
        email: '',
        password: '',
        confirmPassword: '',
        logo: ''
      },
      errors: {
        name: '',
        email: '',
        password: '',
        confirmPassword: '',
        logo: ''
      },
      export_button: true,
      export_file: '',
      actives: ["Active", "Inactive"]
    }
  },
  mounted() {
    setTimeout(function () {
      $(".VueTables__search").parent().remove();
    });
    this.companyUserListing();
  },
  methods: {
    companyUserListing(){
      this.bus.$on('callUserListings',  () => {
        this.dataTable.reloadProp = !this.dataTable.reloadProp;
      });
    },
    companySearch() {
      console.log('call', this.companyUserObj.search);
      this.dataTable.searchProp = this.companyUserObj.search;
    },
    segmentAction(rowObj) {
      $(".table_drop:not(#" + rowObj.row.id + "_action)").hide();
      $("#" + rowObj.row.id + "_action").slideToggle('100');
    },
    removeActive(id) {
      console.log('remove', id);
      $("#" + id).removeClass("active");
      this.dataTable.lookupProp = {};
    },
    fn_userFilter(label, index) {
      $(".sideBar-filter").removeClass("active");
      $("#active" + index).addClass("active");
      console.log('label', label);
      this.dataTable.lookupProp = label;
    },
    deleteUsers(id) {
      this.$swal({
        title: "Are you sure? You want to delete this !",
        type: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        confirmButtonText: "Yes, Delete it!"
      }).then((result) => {
        if (result.value) {
          this.api("post",
              this.constants.getUrl("backendApiUrl"),
              {
                resource: "bounced/user/delete/" + id,
                action: "delete",
                data: {
                  id: id
                }
              },
              this.authHeaders
          ).then((response) => {
            this.$swal('Deleted', 'You successfully deleted this file', 'success');
            this.dataTable.reloadProp = !this.dataTable.reloadProp;
          }).catch((error) => {
            console.log('error', error);
          });
        }
      });
    }
  }
}
</script>

<style>
</style>