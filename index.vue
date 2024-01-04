<template>
  <div class="container">
    <div v-if="isLoading" class="flex-middle justifycontent-center">
      <spinner :show="isLoading" size="50"></spinner>
    </div>
    <div v-else-if="selectedField.fieldValue === null" class="empty-state-handling">
      <img src="~assets/img/workorder.svg" />
      <div class="fc-black-14 pT10">
        Please select site field in form.
      </div>
    </div>
    <div class="p10" v-else>
      <div class>
        <div v-if="selectedField.fieldName === 'unit'">
          <el-tag class="unit-tag" v-if="selectedField.data.name" style="margin-top: 5px; margin-bottom: 5px;">{{ selectedField.data.name }}</el-tag>
          <el-row class="pT10">
            <el-col :span="3" v-if="selectedField.data.tenant && selectedField.data.tenant.name">
              <div
                class="fc-avatar"
              >{{ selectedField.data.tenant ? getAvatarName(selectedField.data.tenant.name) : '' }}</div>
            </el-col>
            <el-col :span="20" v-if="selectedField.data.tenant">
              <div
                class="fc-black-15 fw5"
                v-if="selectedField.data.tenant && selectedField.data.tenant.name"
              >{{selectedField.data.tenant ? selectedField.data.tenant.name : ''}}</div>
              <div class="flex-middle flex-wrap mT5" v-if="selectedField.data.tenant.primaryContactEmail || selectedField.data.tenant.primaryContactPhone">
                <div
                  class="fc-blue13"
                  v-if="selectedField.data.tenant.primaryContactEmail"
                ><i class="el-icon-message mR5" style="color:#3ab2c2;"></i>{{selectedField.data.tenant.primaryContactEmail}}</div>
                <el-divider direction="vertical" class="mL10 mR10" style="background-color:#ecedf4;" v-if="selectedField.data.tenant.primaryContactEmail && selectedField.data.tenant.primaryContactPhone"></el-divider>
                <div
                  class="fc-blue13"
                  v-if="selectedField.data.tenant.primaryContactPhone"
                ><i class="el-icon-mobile-phone mR5" style="color:#3ab2c2;"></i>{{selectedField.data.tenant.primaryContactPhone}}</div>
                <div style="width: 100%; height:0px;"></div>
                <div class="flex-middle flex-wrap mT5" v-if="getBasespaceLocation(selectedField.data)">
                <i class="el-icon-location-outline mR5" style="color:#3ab2c2;"></i>
                <div
                  class="fc-blue13"
                  v-for="(loc, index) in getBasespaceLocation(selectedField.data)" :key="index">
                  <i class="el-icon-arrow-right" style="margin-left: 5px; opacity: 0.7;" v-if="index != 0"></i>
                  {{loc}}
                  </div>
              </div>
              <div style="width: 100%; height:0px;"></div>
              <div class="flex-middle flex-wrap mT5" v-if="getResidentType(selectedField.data)">
                <i class="el-icon-house mR5" style="color:#3ab2c2;"></i>
                <div class="fc-blue13">
                  {{ getResidentType(selectedField.data).label }}
                </div>
              </div>
              </div>
            </el-col>
          </el-row>
        </div>
        <div v-if="selectedField.fieldName === 'resource' && selectedField.data.type === 'basespace'">
          <el-tag class="unit-tag" style="margin-top: 5px; margin-bottom: 5px;">{{ selectedField.data.record.spaceTypeEnum }}</el-tag>
          <el-row class="pT10">
            <el-col :span="3" v-if="selectedField.data.record && selectedField.data.record.name">
              <div
                class="fc-avatar"
              >{{ getAvatarName(selectedField.data.record.name) }}</div>
            </el-col>
            <el-col :span="20">
              <div
                class="fc-black-15 fw5"
                v-if="selectedField.data.record && selectedField.data.record.name"
              >{{selectedField.data.record.name}}</div>
              <div class="flex-middle flex-wrap mT5" v-if="getBasespaceLocation(selectedField.data.record)">
                <i class="el-icon-location-outline mR5" style="color:#3ab2c2;"></i>
                <div
                  class="fc-blue13"
                  v-for="(loc, index) in getBasespaceLocation(selectedField.data.record)" :key="index">
                  <el-divider direction="vertical" style="background-color:#ecedf4; margin: 0 8px;" v-if="index != 0"></el-divider>
                  {{loc}}
                  </div>
              </div>
            </el-col>
          </el-row>
        </div>
        <div v-if="selectedField.fieldName === 'resource' && selectedField.data.type === 'asset'">
          <el-tag class="unit-tag" style="margin-top: 5px; margin-bottom: 5px;">{{ selectedField.data.record.resourceTypeEnum }}</el-tag>
          <el-row class="pT10">
            <el-col :span="3" v-if="selectedField.data.record && selectedField.data.record.name">
              <div
                class="fc-avatar"
              >{{ getAvatarName(selectedField.data.record.name) }}</div>
            </el-col>
            <el-col :span="20">
              <div
                class="fc-black-15 fw5"
                v-if="selectedField.data.record && selectedField.data.record.name"
              >{{selectedField.data.record.name}}</div>
              <div class="flex-middle flex-wrap mT5" v-if="selectedField.data.record.space && getBasespaceLocation(selectedField.data.record.space)">
                <i class="el-icon-location-outline mR5" style="color:#3ab2c2;"></i>
                <div
                  class="fc-blue13"
                  v-for="(loc, index) in getBasespaceLocation(selectedField.data.record.space)" :key="index">
                  <el-divider direction="vertical" style="background-color:#ecedf4; margin: 0 8px;" v-if="index != 0"></el-divider>
                  {{loc}}
                  </div>
              </div>
            </el-col>
          </el-row>
        </div>
        <div v-if="isLoading" class="flex-middle justifycontent-center">
          <spinner :show="isLoading" size="50"></spinner>
        </div>
        <div v-else-if="noWorkorders" class="empty-state-handling">
          <img src="~assets/img/workorder.svg" />
          <div class="fc-black-14 pT10">No recent workorders found in this {{selectedField.fieldName}}</div>
        </div>
        <div class="fc-recent-wo-con" v-else>
           <div class="flex-middle justifycontent-space">
            <div class="fc-black-12 text-uppercase fw5 text-after-border" style="font-size: 12px;">
            RECENT WORKORDERS
          </div>
          <div class="text-center fc-id fc-hover-underline text-uppercase pointer" @click="openWoList()" v-if="recentWorkorders.length >= 2">
              View All
            </div>
          </div>
          <el-divider class="fc-border-pink"></el-divider>
          <div class="fc-wo-details-con">
            <div class v-for="(workorder, index) in recentWorkorders" :key="index">
              <div class="pointer fc-card-hover">
                <div class="flex-middle">
                  <div class="fc-id" @click="openSummary(workorder)">#{{workorder.id}}</div>
                  <el-divider direction="vertical" class="mL10 mR10 fc-divider-grey"></el-divider>
                  <div class="fc-black-12 text-uppercase">{{workorder.status.displayName}}</div>
                </div>
                <div class="fc-black-14 fw5 pT10">{{workorder.subject}}</div>
                <div class="flex-middle pT10" style="padding-top: 8px;">
                  <i class="el-icon-time" style="color: #8ca1ad;font-size: 14px;"></i>
                  <div class="fc-black-13 pL10" style="font-size: 12px;">{{formatDate(workorder.createdTime)}}</div>
                </div>
                <el-button type="primary" plain size="mini" class="create-call-log-btn" @click="createCallLog(workorder)">Create Call Log</el-button>
                <el-divider class="fc-divider-grey fc-divider-bottom "></el-divider>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import spinner from "../../components/spinner.vue";
import moment from "moment-timezone";
export default {
  data() {
    return {
      recentWorkorders: [],
      noWorkorders: false,
      isLoading: false,
      selectedField: {
        fieldName: null,
        fieldValue: null,
        data: null
      },
      currentOrg: null,
      residentType: null
    };
  },
  components: {
    spinner
  },
  mounted() {
    window.app = FacilioAppSDK.init();
    this.init();
  },
  methods: {
    init() {
      window.app.on("app.loaded", data => {
        // console.log('##########: ', data)
        this.currentOrg = window.app.getCurrentOrg();
        if (this.currentOrg && this.currentOrg.domain == 'renaissance') {
          this.loadResidentType()
        }
        window.app.on("form.siteId.changed", data => {
            console.log('########## site field changed ', data.value)
          this.selectedField.fieldName = 'siteId';
          this.selectedField.fieldValue = data;
          if (data && data.value && data.value.id) {
            this.getWorkorderList();
          } else {
              this.recentWorkorders = [null]
          }
        });
      });
    },
    createCallLog(workorder) {
      window.app.interface.trigger('getFormData').then((data) => {
        console.log('form data: ',data)

        let queryParam = {};
        queryParam.work_order_custom_calllog = workorder.id

        if (data.inboundway_workorder != null && data.inboundway_workorder.id != null) {
          queryParam.source_custom_calllog = data.inboundway_workorder.id
        }
        if (data.custom_site_workorder != null && data.custom_site_workorder.id != null) {
          queryParam.site_of_client_custom_calllog = data.custom_site_workorder.id
        }
        if (data.email_id_workorder != null) {
          queryParam.e_maild_id_custom_calllog = data.email_id_workorder
        }
        if (data.e_mail_sender_workorder != null && data.e_mail_sender_workorder.id != null) {
          queryParam.e_mail_sender_custom_calllog_2 = data.e_mail_sender_workorder.id
        }
        if (data.person_in_call_workorder_1 != null && data.person_in_call_workorder_1.id != null) {
          queryParam.person_in_call_custom_calllog = data.person_in_call_workorder_1.id
        }
        if (data.caller_name_workorder_1 != null) {
          queryParam.caller_name_custom_calllog = data.caller_name_workorder_1
        }
        if (data.phone_number_of_site_personel_workorder != null) {
          queryParam.phone_number_of_site_personel_custom_calllog = data.phone_number_of_site_personel_workorder
        }
        if (data.email_sender_name_workorder != null) {
          queryParam.e_mail_sender_name_custom_calllog = data.email_sender_name_workorder
        }
        if (data.cmms_id_workorder_1 != null) {
          queryParam.cmms_id_custom_calllog = data.cmms_id_workorder_1
        }
        if (data["WorkOrder Comments_workorder"] != null) {
          queryParam.mult = data["WorkOrder Comments_workorder"]
        }
        if (data.subject != null) {
          queryParam.name = data.subject
        }

         window.parent.location.href = 'https://app.facilio.com/tutenlabs/work-order/call-log/new' + this.jsonToQueryString(queryParam);
      })
    },
    jsonToQueryString(json) {
      return '?' + 
          Object.keys(json).map(function(key) {
              return key + '=' +
                  encodeURIComponent(json[key]);
          }).join('&')
    },
    getWorkorderList() {
      this.isLoading = true;
      let filters = `{"${this.selectedField.fieldName}":{"operatorId":36,"value":["${this.selectedField.fieldValue.value.id}"]}}`;
      let url = `/workorder/all?page=1&perPage=5&filters=${encodeURIComponent(
        filters
      )}&viewName=all&includeParentFilter=true&orderBy=createdTime&orderType=desc`;
      let options = {
        method: "GET"
      };
      this.recentWorkorders = [];
      this.noWorkorders = false;
      window.app.request
        .invokeFacilioAPI(url, options)
        .then(response => {
          this.recentWorkorders = response && response.workOrders;
          if (!this.recentWorkorders || !this.recentWorkorders.length) {
            this.noWorkorders = true;
          }
          this.isLoading = false;
        })
        .catch(error => {
          this.isLoading = false;
        });
    },
    getTenantUnit() {
      this.isLoading = true;
      let url = `/v2/module/data/${this.selectedField.fieldValue.value.id}?moduleName=tenantunit`;
      let options = {
        method: "GET"
      };
      window.app.request
        .invokeFacilioAPI(url, options)
        .then(response => {
          this.selectedField.data =
            response && response.result && response.result.moduleData;
          this.isLoading = false;
        })
        .catch(error => {
          this.isLoading = false;
          // error in request
        });
    },
    getResource() {
      this.isLoading = true;
      let url = `/v2/basespaces/${this.selectedField.fieldValue.value.id}`;
      let options = {
        method: "GET"
      };
      window.app.request
        .invokeFacilioAPI(url, options)
        .then(response => {
          if (response && response.result && response.result.basespace) {
            this.selectedField.data = {
              type: 'basespace',
              record: response.result.basespace
            }
            this.isLoading = false;
          }
          else {
            let url = `/v2/assets/${this.selectedField.fieldValue.value.id}?fetchHierarchy=true`;
            let options = {
              method: "GET"
            };
            window.app.request
              .invokeFacilioAPI(url, options)
              .then(response => {
                if (response && response.result && response.result.asset) {
                  this.selectedField.data = {
                    type: 'asset',
                    record: response.result.asset
                  }
                  this.isLoading = false;
                }
                else {
                  this.isLoading = false;
                }
              })
              .catch(error => {
                this.isLoading = false;
                // error in request
              });
          }
        })
        .catch(error => {
          this.isLoading = false;
          // error in request
        });
    },
    formatDate(date, excludeTime, onlyTime) {
      if (this.currentOrg) {
        let timezone = this.currentOrg.timezone;
        let dateformat = this.currentOrg.dateFormat || "DD-MMM-YYYY";
        let timeformat = this.currentOrg.timeFormat === 2 ? "hh:mm A" : "HH:mm";

        if (onlyTime) {
          return moment(date)
            .tz(timezone)
            .format(timeformat);
        } else if (excludeTime) {
          return moment(date)
            .tz(timezone)
            .format(dateformat);
        } else {
          return moment(date)
            .tz(timezone)
            .format(dateformat + " " + timeformat);
        }
      }
      return moment(date)
        .tz(timezone)
        .format("LLL");
    },
    getAvatarName(name) {
      var parts = name.split(/[ -]/);
      var initials = "";
      var initialLen = 1;
      var count = 0;
      for (var i = 0; i < parts.length; i++) {
        if (parts[i].trim() !== "") {
          initials += parts[i].charAt(0);
          count++;
          if (count >= initialLen) {
            break;
          }
        }
      }

      if (initials.length < initialLen && name.length >= initialLen) {
        initials = name.trim().substring(0, initialLen);
      }
      var avatarName = initials.toUpperCase();
      return avatarName;
    },
    getBasespaceLocation(basespace) {
      let spaceLocation = [];
      if (basespace.site && basespace.site.id !== basespace.id) {
        spaceLocation.push(basespace.site.name);
      }
      if (basespace.building && basespace.building.id !== basespace.id) {
        spaceLocation.push(basespace.building.name);
      }
      if (basespace.floor && basespace.floor.id !== basespace.id) {
        spaceLocation.push(basespace.floor.name);
      }
      if (basespace.space && basespace.space.id !== basespace.id) {
        spaceLocation.push(basespace.space.name);
      }
      if (basespace.space1 && basespace.space1.id !== basespace.id) {
        spaceLocation.push(basespace.space1.name);
      }
      if (basespace.space2 && basespace.space2.id !== basespace.id) {
        spaceLocation.push(basespace.space2.name);
      }
      if (basespace.space3 && basespace.space3.id !== basespace.id) {
        spaceLocation.push(basespace.space3.name);
      }
      if (basespace.space4 && basespace.space4.id !== basespace.id) {
        spaceLocation.push(basespace.space4.name);
      }
      if (!spaceLocation.length) {
        return null
      }
      return spaceLocation;
    },
    loadResidentType() {
      app.request.invokeFacilioAPI('/v3/picklist/custom_residencetype')
        .then((response) => {
          if (response && response.data && response.data.pickList) {
            this.residentType = response.data.pickList
          }
        })
    },
    getResidentType(tenantunit) {
      if (tenantunit && tenantunit.building && tenantunit.building.data && tenantunit.building.data.residencetype) {
        if (this.residentType) {
          return this.residentType.find((r) => r.value == tenantunit.building.data.residencetype.id)
        }
      }
      if (tenantunit && tenantunit.space1 && tenantunit.space1.data && tenantunit.space1.data.residencetype) {
        if (this.residentType) {
          return this.residentType.find((r) => r.value == tenantunit.space1.data.residencetype.id)
        }
      }
      return null
    },
    openSummary(workorder) {
      let options = {
        module: "workorder",
        id: workorder.id
      };
      app.interface.openSummary(options, true);
    },
    openWoList() {
      let filters = {};
      filters[this.selectedField.fieldName] = {
        operatorId: 36,
        value: [this.selectedField.fieldValue.value.id + ""]
      }
      let options = {
        module: "workorder",
        view: "all",
        filters: filters
      };
      app.interface.openListView(options, true);
    },
    full() {
      window.app.interface
        .trigger("setValue", {
          fieldName: "unit",
          value: 110
        })
        .then(data => console.log("setvalue", data));
      let value = window.app.interface
        .trigger("getValue", {
          fieldName: "unit"
        })
        .then(data => console.log("getvalue", data));
    }
  }
};
</script>
<style>
.create-call-log-btn {
  font-weight: bold;
  float: right;
  position: relative;
  bottom: 48px;
}
</style>