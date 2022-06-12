<template>
  <div class="layout-px-spacing">
    <portal to="breadcrumb">
      <ul class="navbar-nav flex-row">
        <li>
          <div class="page-header">
            <nav class="breadcrumb-one" aria-label="breadcrumb">
              <ol class="breadcrumb">
                <li class="breadcrumb-item active" aria-current="page">
                  <span>Instructor - Announcement</span>
                </li>
              </ol>
            </nav>
          </div>
        </li>
      </ul>
    </portal>

    <div class="row layout-top-spacing">
      <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
        <div class="seperator-header">
          <h4>View Announcement List in Section {{ form.sectionName }} and Subject {{ form.subjectName }}</h4>
        </div>
        <div class="panel br-6 p-0">
          <div class="custom-table">
            <div class="d-flex flex-wrap justify-content-center justify-content-sm-start px-3 pt-3 pb-0">
              <b-button variant="primary" class="m-1" v-b-modal.announcementModal> Add Announcement </b-button>
            </div>
            <div class="table-header">
              <div class="d-flex align-items-center">
                <span>Results :</span>
                <span class="ml-2">
                  <b-select v-model="table_option.page_size" size="sm">
                    <b-select-option value="5">5</b-select-option>
                    <b-select-option value="10">10</b-select-option>
                    <b-select-option value="20">20</b-select-option>
                    <b-select-option value="50">50</b-select-option>
                  </b-select>
                </span>
              </div>
              <div class="header-search">
                <b-input v-model="table_option.search_text" size="sm" placeholder="Search..." />
                <div class="search-image">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    class="feather feather-search"
                  >
                    <circle cx="11" cy="11" r="8"></circle>
                    <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                  </svg>
                </div>
              </div>
            </div>

            <b-table
              ref="basic_table"
              responsive
              hover
              :items="items"
              :fields="columns"
              :per-page="table_option.page_size"
              :current-page="table_option.current_page"
              :filter="table_option.search_text"
              sort-by="name"
              :show-empty="true"
              @filtered="on_filtered"
            >
              <template #cell(date)="data">
                <span>{{ moment.utc(data.item.deadline).format('MMMM D, YYYY') }}</span>
              </template>
              <template #cell(time)="data">
                <span>{{ moment.utc(data.item.deadline).format('h:mm A') }}</span>
              </template>
              <template #cell(action)="data">
                <b-button size="sm mr-3" pill variant="dark" v-b-modal.announcementEditModal @click="editAnnouncement(data.item.id)">Edit</b-button>
                <b-button size="sm" pill variant="danger" @click="deleteAnnouncement(data.item.id)">Delete</b-button>
              </template>
            </b-table>

            <div class="table-footer">
              <div class="dataTables_info">Showing {{ meta.total_items ? meta.start_index + 1 : 0 }} to {{ meta.end_index + 1 }} of {{ meta.total_items }}</div>
              <div class="paginating-container pagination-solid flex-column align-items-right">
                <b-pagination
                  v-model="table_option.current_page"
                  :total-rows="table_option.total_rows"
                  :per-page="table_option.page_size"
                  prev-text="Prev"
                  next-text="Next"
                  first-text="First"
                  last-text="Last"
                  first-class="first"
                  prev-class="prev"
                  next-class="next"
                  last-class="last"
                  class="rounded"
                >
                  <template #first-text>
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7" />
                    </svg>
                  </template>
                  <template #prev-text>
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                    </svg>
                  </template>
                  <template #next-text>
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                    </svg>
                  </template>
                  <template #last-text>
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7" />
                    </svg>
                  </template>
                </b-pagination>
              </div>
            </div>
          </div>
        </div>
      </div>
      <vue-progress-bar></vue-progress-bar>
    </div>
    <!--Add Modal for Announcement -->
    <b-modal id="announcementModal" title="Add Announcement" centered hide-footer no-close-on-backdrop @hidden="announcementResetModal">
      <span class="form-text text-muted mb-2"
        ><span style="color: #009688">Note:</span> Unless you click the cancel or "X" button, this window will not close. Otherwise, clicking one of those two will dismiss the window and reset
        everything. Please take caution.</span
      >
      <b-form action="#" @submit.prevent="addAnnouncement" @keydown="errors.clear($event.target.name)">
        <b-form-group label="Date and Time">
          <flat-pickr
            @on-change="clearDateError"
            v-model="form.deadline"
            :config="{ minDate: 'today', static: true, wrap: true, enableTime: true, dateFormat: 'Y-m-d H:i:S' }"
            placeholder="Select Date and Time"
            class="form-control flatpickr active"
          ></flat-pickr>
          <span class="text-danger" v-text="errors.get('deadline')"></span>
        </b-form-group>
        <b-form-group label="Activity Title (URL)">
          <b-input v-model="form.act_title" name="act_title" type="text" placeholder="Activity Title"></b-input>
          <span class="text-danger" v-text="errors.get('act_title')"></span>
        </b-form-group>
        <b-form-group label="Instruction">
          <b-textarea rows="5" v-model="form.instruction" name="instruction" placeholder="Instruction"></b-textarea>
          <span class="text-danger" v-text="errors.get('instruction')"></span>
        </b-form-group>
        <b-form-group label="Activity Link">
          <b-input v-model="form.act_link" name="act_link" type="text" placeholder="Activity Link"></b-input>
          <span class="text-danger" v-text="errors.get('act_link')"></span>
        </b-form-group>
        <b-form-group label="Attachment (URL) (Optional)">
          <b-input v-model="form.attachment" name="attachment" type="text" placeholder="Attachment (Optional)"></b-input>
          <span class="text-danger" v-text="errors.get('attachment')"></span>
        </b-form-group>
        <hr />
        <div class="d-flex flex-wrap justify-content-center justify-content-sm-end">
          <b-button type="submit" variant="primary" class="mt-3 m-1">Create</b-button>
          <b-button variant="danger" class="mt-3 m-1" @click="$bvModal.hide('announcementModal')">Cancel</b-button>
        </div>
      </b-form>
    </b-modal>
    <!--Edit Modal for Announcement -->
    <b-modal id="announcementEditModal" title="Edit Announcement" centered hide-footer @hidden="announcementResetModal">
      <b-form action="#" @submit.prevent="updateAnnouncement" @keydown="errors.clear($event.target.name)">
        <b-form-group label="Date and Time">
          <flat-pickr
            v-model="form.deadline"
            :config="{ static: true, wrap: true, enableTime: true, dateFormat: 'Y-m-d H:i:S' }"
            placeholder="Select Date and Time"
            class="form-control flatpickr active"
          ></flat-pickr>
          <span class="text-danger" v-text="errors.get('deadline')"></span>
        </b-form-group>
        <b-form-group label="Activity Title">
          <b-input v-model="form.act_title" name="act_title" type="text" placeholder="Activity Title"></b-input>
          <span class="text-danger" v-text="errors.get('act_title')"></span>
        </b-form-group>
        <b-form-group label="Instruction">
          <b-textarea rows="5" v-model="form.instruction" name="instruction" placeholder="Instruction"></b-textarea>
          <span class="text-danger" v-text="errors.get('instruction')"></span>
        </b-form-group>
        <b-form-group label="Activity Link (URL)">
          <b-input v-model="form.act_link" name="act_link" type="text" placeholder="Activity Link"></b-input>
          <span class="text-danger" v-text="errors.get('act_link')"></span>
        </b-form-group>
        <b-form-group label="Attachment (URL) (Optional)">
          <b-input v-model="form.attachment" name="attachment" type="text" placeholder="Attachment (Optional)"></b-input>
          <span class="text-danger" v-text="errors.get('attachment')"></span>
        </b-form-group>
        <hr />
        <div class="d-flex flex-wrap justify-content-center justify-content-sm-end">
          <b-button type="submit" variant="primary" class="mt-3 m-1">Update</b-button>
          <b-button variant="danger" class="mt-3 m-1" @click="$bvModal.hide('announcementEditModal')">Cancel</b-button>
        </div>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
//flatpickr
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';
import '@/assets/sass/forms/custom-flatpickr.css';
import Vue from 'vue';
import VueMask from 'v-mask';
Vue.use(VueMask);
import Errors from '@/main.js';
export default {
  metaInfo: { title: 'Announcement' },
  components: {
    flatPickr,
  },
  data() {
    return {
      form: {
        id: '',
        deadline: '',
        act_title: '',
        instruction: '',
        act_link: '',
        attachment: '',
        sectionName: '',
        subjectName: '',
        sectionID: this.$route.params.section_id,
        subjectID: this.$route.params.subject_id,
      },
      errors: new Errors(),
      items: [],
      columns: [],
      table_option: { total_rows: 0, current_page: 1, page_size: 10, search_text: '' },
      meta: {},
    };
  },
  watch: {
    table_option: {
      handler: function () {
        this.get_meta();
      },
      deep: true,
    },
  },
  mounted() {
    this.bind_data();
  },
  created() {
    // console.log(id);
    this.$http
      .get(`/api/assigned/info/${this.form.sectionID}/${this.form.subjectID}`, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`,
        },
      })
      .then((response) => {
        this.form.sectionName = response.data.data.section.section;
        this.form.subjectName = response.data.data.subject.subject;
      })
      .catch(() => {});
  },
  methods: {
    bind_data() {
      this.columns = [
        { key: 'date', label: 'Due Date' },
        { key: 'time', label: 'Closing Time' },
        { key: 'act_title', label: 'Activity Title' },
        { key: 'action', label: 'Actions', class: 'actions text-center' },
      ];
      let fetchTodo = async () => {
        this.items = [];
        if (this.$route.params.section_id == 'undefined') {
          this.$swal.fire('Not Found!', 'Please click the announcement button in section page.', 'warning');
          this.$router.push({ name: 'instructorAssign' });
        } else {
          this.$http
            .get(`/api/announcement/${this.form.sectionID}/${this.form.subjectID}`, {
              headers: {
                Authorization: `Bearer ${localStorage.getItem('token')}`,
              },
            })
            .then((res) => {
              this.items = res.data.data;
              this.table_option.total_rows = this.items.length;
              this.get_meta();
            })
            .catch((errors) => {
              console.log(errors);
            });
        }
      };

      fetchTodo();
    },
    addAnnouncement() {
      this.$Progress.start();
      this.$http
        .post(`/api/announcement/${this.form.sectionID}/${this.form.subjectID}`, this.form, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then(() => {
          this.$toaster.success('Announcement Created Successfuly!');
          this.$nextTick(() => {
            this.$bvModal.hide('announcementModal');
          });
          this.bind_data();
          this.$Progress.finish();
          // this.$http
          //   .get(process.env.VUE_APP_CHATBOT_URL + 'brodcast-announcement/' + localStorage.getItem('token') + '/' + this.form.sectionID + '/' + this.form.subjectID)
          //   .then(() => {
          //     this.$swal.fire('Success!', 'Announcement broadcasted successfully!', 'success');
          //   })
          //   .catch((errors) => {
          //     this.errors.record(errors.response.data.errors);
          //   });
        })
        .catch((errors) => {
          this.errors.record(errors.response.data.errors);
          this.$Progress.fail();
        });
    },
    editAnnouncement(id) {
      this.$http
        .get('/api/announcement/' + id, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          this.form.id = response.data.data.id;
          this.form.deadline = response.data.data.deadline;
          this.form.act_title = response.data.data.act_title;
          this.form.instruction = response.data.data.instruction;
          this.form.act_link = response.data.data.act_link;
          this.form.attachment = response.data.data.attachment;
        })
        .catch((errors) => {
          this.errors.record(errors.response.data.errors);
        });
    },
    deleteAnnouncement(id) {
      this.$swal
        .fire({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          icon: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes, delete it!',
        })
        .then((result) => {
          if (result.isConfirmed) {
            //Send delete request
            this.$Progress.start();
            this.$http
              .delete('/api/announcement/' + id, {
                headers: {
                  Authorization: `Bearer ${localStorage.getItem('token')}`,
                },
              })
              .then(() => {
                this.$swal.fire('Deleted!', 'Announcement has been deleted.', 'success');
                this.bind_data();
                this.$Progress.finish();
              })
              .catch(() => {
                this.$swal.fire('Failed!', 'There was something wrong.', 'warning');
                this.$Progress.fail();
              });
          }
        });
    },
    updateAnnouncement() {
      var id = this.form.id;
      let self = this;
      var axios = require('axios');
      var data = this.form;
      var config = {
        method: 'put',
        url: '/api/announcement/' + id,
        headers: {
          Authorization: 'Bearer ' + localStorage.getItem('token'),
        },
        data: data,
      };
      self.$Progress.start();
      axios(config)
        .then(function () {
          self.$toaster.success('Announcement Update Successfuly!');
          self.$nextTick(() => {
            self.$bvModal.hide('announcementEditModal');
          });
          self.bind_data();
          self.$Progress.finish();
        })
        .catch(function (errors) {
          self.errors.record(errors.response.data.errors);
          self.$Progress.fail();
        });
    },
    clearDateError() {
      this.errors.clear('deadline');
    },
    announcementResetModal() {
      this.form.deadline = 'deadline';
      this.form.deadline = '';
      this.errors.clear('deadline');

      this.form.act_title = 'act_title';
      this.form.act_title = '';
      this.errors.clear('act_title');

      this.form.instruction = 'instruction';
      this.form.instruction = '';
      this.errors.clear('instruction');

      this.form.act_link = 'act_link';
      this.form.act_link = '';
      this.errors.clear('act_link');

      this.form.attachment = 'attachment';
      this.form.attachment = '';
      this.errors.clear('attachment');
    },
    on_filtered(filtered_items) {
      this.refresh_table(filtered_items.length);
    },
    refresh_table(total) {
      this.table_option.total_rows = total;
      this.table_option.currentPage = 1;
    },
    get_meta() {
      var startPage;
      var endPage;
      var totalPages = this.table_option.page_size < 1 ? 1 : Math.ceil(this.table_option.total_rows / this.table_option.page_size);
      totalPages = Math.max(totalPages || 0, 1);

      var maxSize = 5;
      var isMaxSized = typeof maxSize !== 'undefined' && maxSize < totalPages;
      if (isMaxSized) {
        startPage = Math.max(this.table_option.current_page - Math.floor(maxSize / 2), 1);
        endPage = startPage + maxSize - 1;

        if (endPage > totalPages) {
          endPage = totalPages;
          startPage = endPage - maxSize + 1;
        }
      } else {
        startPage = 1;
        endPage = totalPages;
      }
      let startIndex = (this.table_option.current_page - 1) * this.table_option.page_size;
      let endIndex = Math.min(startIndex + this.table_option.page_size - 1, this.table_option.total_rows - 1);

      var pages = Array.from(Array(endPage + 1 - startPage).keys()).map((i) => startPage + i);
      this.meta = {
        total_items: this.table_option.total_rows,
        current_page: this.table_option.current_page,
        page_size: this.table_option.page_size,
        total_pages: totalPages,
        start_page: startPage,
        end_page: endPage,
        start_index: startIndex,
        end_index: endIndex,
        pages: pages,
      };
    },
  },
};
</script>
