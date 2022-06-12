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
          <h4>View Announcement List</h4>
        </div>
        <div class="panel br-6 p-0">
          <div class="custom-table">
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
              <template #cell(section)="data">
                <span>{{ data.item.section.section }}</span>
              </template>
              <template #cell(subject)="data">
                <span>{{ data.item.subject.subject }}</span>
              </template>
              <template #cell(instructor)="data">
                <span>{{ data.item.instructor.first_name }} {{ data.item.instructor.last_name }}</span>
              </template>
              <template #cell(status)="data">
                <b-badge variant="success" v-if="data.item.status == 'Complete'"> Complete </b-badge>
                <b-badge variant="danger" v-else> Incomplete </b-badge>
              </template>
              <template #cell(action)="data">
                <b-button size="sm mr-3" pill variant="secondary" v-b-modal.announcementModal @click="announcementDetails(data.item.id)">Details</b-button>

                <b-button size="sm" pill variant="dark" v-if="data.item.status == 'Complete'" @click="incompleteAnnouncement(data.item.id)">Incomplete</b-button>
                <b-button size="sm" pill variant="primary" v-else @click="completeAnnouncement(data.item.id)">Complete</b-button>
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
    <!--Modal for Announcement Details-->
    <b-modal id="announcementModal" title="Announcement Details" centered hide-footer>
      <p>
        <span v-if="form.current_status == 'Complete'" style="color: #009688">Status: <b-badge variant="success"> Complete </b-badge></span>
        <span v-else style="color: #009688">Status: <b-badge variant="danger"> Incomplete </b-badge></span>
      </p>
      <p><span style="color: #009688">Due Date:</span> {{ moment.utc(form.deadline).format('MMMM D, YYYY') }}</p>
      <p>
        <span style="color: #009688">Closing Time:</span> <span>{{ moment.utc(form.deadline).format('h:mm A') }}</span>
      </p>
      <p>
        <span style="color: #009688">Activity Title:</span> <span>{{ form.act_title }}</span>
      </p>
      <p>
        <span style="color: #009688">Instruction:</span>
      </p>
      <p style="white-space: pre-line">
        {{ form.instruction }}
      </p>
      <p>
        <span style="color: #009688">Activity Link:</span><span>{{ form.act_link }}</span>
      </p>
      <p>
        <span style="color: #009688">Attachment:</span><span>{{ form.attachment }}</span>
      </p>

      <div class="d-flex flex-wrap justify-content-center justify-content-sm-end">
        <b-button variant="danger" class="mt-3 m-1" @click="$bvModal.hide('announcementModal')">Close</b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>
import Errors from '@/main.js';
export default {
  metaInfo: { title: 'Announcement' },
  data() {
    return {
      form: {
        id: '',
        deadline: '',
        act_title: '',
        instruction: '',
        act_link: '',
        attachment: '',
        section: '',
        subject: '',
        instructor: '',
        current_status: '',
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
  methods: {
    bind_data() {
      this.columns = [
        { key: 'date', label: 'Due Date' },
        { key: 'time', label: 'Closing Time' },
        { key: 'act_title', label: 'Activity Title' },
        { key: 'section', label: 'Section' },
        { key: 'subject', label: 'Subject' },
        { key: 'instructor', label: 'Instructor' },
        { key: 'status', label: 'Status' },
        { key: 'action', label: 'Actions', class: 'actions text-center' },
      ];
      let fetchTodo = async () => {
        this.items = [];
        this.$http
          .get('/api/student/check/announcement', {
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
      };

      fetchTodo();
    },
    announcementDetails(id) {
      this.$http
        .get('/api/student/check/announcement/' + id, {
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
          this.form.current_status = response.data.status;
        })
        .catch((errors) => {
          this.errors.record(errors.response.data.errors);
        });
    },
    completeAnnouncement(id) {
      let self = this;
      var axios = require('axios');
      var data = this.form;
      var config = {
        method: 'post',
        url: `/api/student/complete/announcement/${id}`,
        headers: {
          Authorization: 'Bearer ' + localStorage.getItem('token'),
        },
        data: data,
      };
      self.$Progress.start();
      axios(config)
        .then(function () {
          self.$toaster.success('Announcement Completed Successfuly!');
          self.bind_data();
          self.$Progress.finish();
        })
        .catch(function (errors) {
          self.errors.record(errors.response.data.errors);
          self.$Progress.fail();
        });
    },
    incompleteAnnouncement(id) {
      this.$Progress.start();
      this.$http
        .delete(`/api/student/incomplete/announcement/${id}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then(() => {
          this.$toaster.success('Announcement Incomplete Successfuly!');
          this.bind_data();
          this.$Progress.finish();
        })
        .catch(() => {
          this.$swal.fire('Failed!', 'There was something wrong.', 'warning');
          this.$Progress.fail();
        });
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
